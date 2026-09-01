# OMS CHIPPY [Beta V1_10_0] — DEVELOPER DOCUMENTATION

**Run date:** 2026-08-31  
**Current Unix Epoch:** 1788134400  
**App Version:** Chippy Beta V1_10_0  
**License:** GPL-3.0  

**Purpose:** The explanatory technical spec for OMS Chippy — how each system works and why. Modeled on
the OMS Dojo Developer Documentation, using the shared OMS **Cortex / Node / Leaf** vocabulary, sized to
Chippy's (younger, simpler) scope.

**This is the "how it works" doc.** Its companion, the **Y System Reference**, is the "where is it" doc:
a machine-readable index of the actual identifiers (constants, nodes, data containers, functions,
schemas). The two do NOT overlap — C explains a system in prose once; Y lists that system's identifiers
once. If you want to understand a system, read C. If you want to find the variable or function, read Y.

> **DOCUMENT TYPE — north star (read before editing).**
> **Developer Documentation** is a *holistic, narrative guide*: it teaches how the software works and
> guides you through architecture, concepts, and workflows. It is contextual and instructional, meant to
> be **read**. That is this document (C).
> Its companion (Y) is a **System Reference**: a precise, dictionary-like listing of the technical
> specifications — identifiers, schemas, parameters — meant for **lookup**, not reading.
> *Adaptation for Chippy:* Chippy is a single-file app with no public API, so Y indexes the **internal
> code identifiers** (the inline-script symbols) rather than a public interface — reference in form,
> internal-code in subject.
> **The editing rule this implies:** all *explanation* (the why, the how, the reasoning, the architecture)
> lives HERE in C. If you're writing a paragraph that teaches, it belongs in C. If you're writing an entry
> that just names a symbol + a one-line "what it is" + a pointer to its C section, it belongs in Y. Keep
> them apart and they never drift into two competing copies again.

---

# 1. INTRODUCTION

## 1.1 What Chippy is
Chippy is one self-contained HTML file — HTML + inline CSS + inline JS, zero external dependencies, no
build step, no network calls, no audio files. Everything (synthesis, rendering, state) runs in the
browser from that one file. This is an OMS-suite-wide principle (shared with Dojo/Sozo): the app must
open and run offline, on any device with a browser (MacBook, iPad, phone, Chromebook), forever, with no
rot. Any change that adds an external dependency (a font server, a framework, a sample library) violates
this and is rejected. All synthesis is generated in code (Web Audio); all fonts are system/web-safe; all
data is in memory.

Chippy is a **generator**, not a player. No recording, no sample library, no fixed song. Every loop is
produced on the fly from a seed + generation rules: same seed → same loop, new seed → a fresh variation.
"Next" and "generate" are the same act (there is no playlist). This is the soul of the app: infinite,
seeded, rule-based music — "anyone can be a OneManShYo."

## 1.2 How to read this doc
- **§2 (Cortex & the critical stack)** — read first. The systems the app cannot function without, in
  dependency order. The Master Clock is the one true Cortex system; everything else in the stack sits on
  top of it. Touch these carefully.
- **§3–§9 (Nodes)** — the feature systems that ride on the Cortex: automation grid, voices, matrix,
  visualizer, title generator. Normal feature work happens here.
- **§10 (Design System)** — the UI/UX governance layer (tokens, type, color, component rules). Important
  and governed, but it is NOT Cortex — breaking it makes the app look wrong, not stop working.
- **§11 (Leaf reference)** — the small controls and keyboard map.
- **§12 (AMU → profile pipeline)** — how Yo DNA is produced upstream (Sozo / omsanalyze).

## 1.3 Cortex / Node / Leaf — what the tiers mean
- **Cortex** = the small, load-bearing core. Everything traces back to it; a change cascades everywhere.
  The label is used **sparingly on purpose** — if everything is Cortex, the word means nothing. In Chippy
  exactly one system is Cortex: **Timing (the Master Clock + scheduler).**
- **Node** = a feature system that depends on the Cortex but that the app's core function survives changes
  to. Most of the app is Nodes.
- **Leaf** = a small control or affordance hanging off a Node.

---

# 2. CORTEX & THE CRITICAL STACK

The Cortex is the timing system. Above it sit three more layers that each depend on the one below —
sound generation, composition, and conditioning. Only the timing layer is *Cortex*; the layers above are
documented here (rather than with the ordinary Nodes) because they form the critical generative spine and
you must understand what each sits on top of.

**The stack, bottom-up (each layer depends on the one beneath it):**

| Layer | What it does | Tier | If it breaks |
|---|---|---|---|
| 1. Timing (Master Clock + scheduler) | places every event in time; drives every visual | **CORTEX** | nothing plays or stays in sync |
| 2. Sound generation (voices/FX synthesis) | turns events into audible sound | critical | no audio at all |
| 3. Composition (genre builders, accents, arrange) | composes the generative music itself | critical | nothing to play — runs even with conditioning off |
| 4. Conditioning (YoConditioner / Yo DNA) | biases composition + mixing toward an identity | rides on top | you lose the *identity*, not the music |

The tell for the ordering: **the YoConditioner sits on top of composition** (turn it off and the genre
builders still compose a full track); **composition sits on top of sound generation** (a pattern is inert
until voices can sound it); **all of it sits on top of timing** (nothing is placed in time without the
clock + scheduler). Timing sits on nothing — that is why it, and only it, is Cortex.

## 2.1 Timing — the Master Clock (CORTEX)

Real-time audio scheduling locked to the AudioContext clock. This is the hardest, most critical system in
the app — browsers were not built for sample-accurate audio — and it is the reason audio and every visual
stay in sync. **Change nothing here without understanding the whole section.**

### 2.1.1 One clock source, two views
There is **one** time source: the browser's audio hardware clock, `actx.currentTime`. Everything derives
from it by subtracting an **anchor** ("since when"). There is no second independent timeline that could
drift. Two anchors give two *views* of the same instant:

- **Session view (continuous).** Anchor = `playOriginCtx`, set once when play is pressed and never moved
  until stop. Answers *how far into the whole set are we?* Counts up forever (1.1.1, 2.1.1 … 47.3.2),
  never wraps. Drives the scrolling roll, the playhead, and the session readout.
- **Loop view (loop-phase).** Anchor = `loopStartCtx`, the audio time the **current loop** began. Answers
  *where are we inside this loop?* Runs 1.1.1 → end-of-loop, then resets to 1.1.1 at the boundary. Drives
  Chippy the conductor, the loop readout, and its last-bar blink.

At play the two anchors are set equal (`playOriginCtx = loopStartCtx`), so both views read `1 . 1 . 1`.
From then on `playOriginCtx` stays put while `loopStartCtx` advances one loop-length per completed loop.
So the loop view is exactly the session view folded at the loop length:

```
loopPosition = sessionPosition mod loopLength
```

They cannot drift relative to each other — same clock, one read modulo the loop. This is the same model a
DAW uses: one transport, shown as continuous song position and position-within-loop.

### 2.1.2 The two accessor functions
- **`sessionClock()` → `{ es, esF, cycles }`** — the continuous view. `es` = whole 16th-steps since play
  (never wraps), `esF` = the same as a smooth fraction (for scrolling), `cycles` = completed loops
  (`floor(es / TOTAL_STEPS)`). Read by the session readout and the roll's continuous-wrap gate.
- **`masterLoopClock()` → `{ loopSec, loopStep, loopFrac, bar, beat, six, L, TOTAL_STEPS }`** — the
  loop-phase view; the single source of truth for everything that resets per loop. Read by Chippy, the
  loop readout, the blink, and the face bob. **Golden rule (from Dojo): every loop-phase consumer reads
  this one function — same formula everywhere, so nothing can drift.**

### 2.1.3 The two rules that make the loop clock correct
`masterLoopClock()` computes loop position as `((actx.currentTime − loopStartCtx) mod L + L) mod L`, with
one guard. Two things must be right or the visuals misbehave:

1. **Modulo-wrap, not clamp-to-zero.** The scheduler advances `loopStartCtx` up to `LOOKAHEAD` (0.25 s)
   *ahead* of audio at a boundary, so `currentTime − loopStartCtx` briefly goes slightly negative right at
   the wrap. Modulo maps that small negative to *near L* (finish the loop's tail) instead of snapping to
   0. Clamping to 0 here is the bug that made Chippy vanish ~250 ms early and the last-bar blink fire early.
2. **Pre-origin holds at 0.** At play `loopStartCtx` is set slightly in the future (see 2.1.5) and audio
   hasn't reached it yet; that case is detected as `currentTime < playOriginCtx` and held at 0, so the loop
   doesn't pre-roll through its tail before any sound.

Note `posSec` (the scheduler-side loop position used for hit detection) still clamps-to-0 by design —
hits must fire on the scheduled grid, not wrap. Visual consumers use `masterLoopClock()`; the audio path
uses `posSec`. The divergence is intentional and per-consumer correct.

### 2.1.4 The scheduler (part of the timing system)
- **`STEPS_PER_BAR = 16`** (16th-note grid); `BARS` = loop length; `TOTAL_STEPS` derived.
- **`LOOKAHEAD = 0.25` s** — the scheduler places events up to this far ahead of `now` each tick, so
  background-tab throttling / GC stutters don't drop notes.
- **`scheduler()`** — the core loop: computes `horizon = now + LOOKAHEAD`, applies pending beat-quantize
  swaps at the next beat, applies pending selecta swaps + length changes at the appropriate boundary, and
  schedules each event via `fireVoice` at `loopStartCtx + ev.t`. It owns every advance of `loopStartCtx`
  (natural wrap, bar-boundary selecta swap, quantized restart).
- **`nextBeatCtx()`** — first beat strictly after `now` (+ a small guard), computed from `playOriginCtx`
  so it's always reachable within the horizon.
- **Persistence pump (`bgPump`)** — a silent node keeping the context alive when the tab is backgrounded
  (Chrome throttles rAF/timers otherwise). Toggled by the ∞ button.

### 2.1.5 Start offset + cold-start warm-up
At play, `loopStartCtx = actx.currentTime + 0.06` — the loop origin is set ~60 ms in the FUTURE so the
first scheduled hits aren't already in the past. Audio routes through a MediaStream `<audio>` bridge
(routable to AirPlay/system output), which takes roughly a beat to spin up on the first play — so the
bridge is WARMED at audio-graph creation (`airEl.play()` + `actx.resume()` inside the first gesture) and
the FIRST start after creation uses a longer 180 ms origin lead (`audioColdStart`). Every subsequent start
uses the normal 60 ms. Both audio and the visual playhead anchor to `loopStartCtx`, so they stay locked.

### 2.1.6 True stop resets the anchors
`stopTransport()` zeroes `posSec` / `prevPos` / `playOriginCtx` / `loopStartCtx` and clears the
pending-quantize queue — a stop is a full reset to bar 1 · beat 1, not a paused resume-position (there is
no resume feature). Leaving the anchors set froze the readout mid-loop.

### 2.1.7 Control-behavior model (how a mid-play change is timed)
Every control that can change mid-play falls into exactly one category, which dictates its wiring:

- **RESTART** (genre) → `quantizeKey()`: waits for the next beat, cleanly restarts the loop. A "new song"
  change.
- **CONTINUE** (tempo, swing, per-voice selectors + levels) → applied LIVE, immediately, no restart (like
  turning a knob).
- **WAIT FOR LOOP** (length / bars) → deferred via `pendingLen`, applied at the loop boundary so the
  current phrase finishes.
- **BAR-BOUNDARY SWAP** (selecta, while playing) → NOT a next-beat restart. Stages the incoming selecta's
  loop (`yoReroll` → `pendingYo`) and arms `skipArmed`; the scheduler swaps it in at the next BAR line via
  `applyPendingSwap`, so the current bar finishes first (the DJ "let the record run out" model — Chippy
  does not overlap/blend). A selecta chosen while STOPPED opens immediately.

When adding a control, classify it into one of these four; that determines its wiring.

## 2.2 Sound generation (voices / FX synthesis)

The layer that turns scheduled events into audible sound. All sound is synthesized — no files.

- **`actx`** — the AudioContext. **`master`** — the master gain node → destination.
- **Oscillator voices** — variable-duty **pulse** waves via `PeriodicWave`, a **triangle** bass, and a
  **noise** channel from a buffer source (LFSR-style). Each voice = oscillator/noise + a short snappy
  envelope. The *timbre* (not the voice count) is what makes it chiptune.
- **Noise buffers** — white `noiseBuf`; pink/brown buffers also generated (used by some FX).
- **Kick synth** — the DRUMS lane is a dedicated pitch-dropping sine + click transient (`playKick`), with
  flavors (909 / 808 / punch / click / noise).
- **FX synths** — `playSweep` (resonant low-pass noise sweep), `playGlitch` (gated stutter), `playHorn`
  (detuned-saw foghorn). See §4.3 for the FX voice.
- **Reverb bus** — a `ConvolverNode` (`reverbBus`) fed by a generated decaying-noise impulse; wet return
  via `reverbWet`. Voices route in per-voice (the module reverb button) and/or via the Master FX bus
  (§4.4). Drums are excluded from the master reverb bus by design (dry kicks).
- **`curDest`** — per-fire output routing: a voice's play function connects to `curDest`, which is
  `master` (dry) or a tap that also feeds `reverbBus` (wet), set at the top of `fireVoice`.

The sound set is deliberately **swappable**: the chiptune voices are the current instrument, but the
layers above (composition, conditioning) address bands, not specific timbres, so a future sound set can
replace this without touching them.

## 2.3 Composition (the generative-music engine)

**How the music is composed** — independent of any conditioning. This layer runs whether or not a selecta
is active; turn the YoConditioner off entirely and Chippy still composes a full track here.

- **`buildPattern(seed)`** — the entry point. Builds a fresh seeded RNG (`mkRnd(seed)`), an empty pattern
  `P` (one array per voice), then calls the active genre builder to fill it. With conditioning off it
  behaves as stock house; with a selecta active it first biases density and plans moves (see §2.4) before
  the same builder runs.
- **Genre builders** — `GENRE[genre].build(P, R)`. **House-only as of V1.8.0** (techno/breaks/dnb/bigroom
  builders are retained in code but unlisted while the conditioner is dialed in on House; they return
  rebuilt to this standard once House is proven).
- **`phraseAccents(P, R)`** — reliably places accents + fills by phrase position.
- **`arrange(P)`** — a long-song arc for manual mode (`BARS ≥ 16` and no selecta); the conditioned set
  keeps the full vibe instead.
- **`rebuildEvents()`** — flattens the pattern `P` into a time-sorted `events[]` (seconds-into-loop `.t`,
  plus the source `step`), which the scheduler then fires.

## 2.4 Conditioning — the YoConditioner + Yo DNA

The layer that biases composition (and the mixing/performance on top of it) toward a real musical
identity. It sits ON TOP of §2.3: it does not compose the music, it tilts how the music is composed and
performed. It applies ONLY when a selecta is active (`yoOn`, i.e. a selecta other than **None**); with
**None** you get raw stock house — neutral gate, no moves.

**Two primitives, kept strictly distinct** (this separation is the whole point — it makes the method
teachable and replicable):

- **Yo DNA — the NOUN.** The committed reference fingerprint of a style. Its *source* can be anything Wes
  officially commits — AMU analysis of tracks (omsanalyze), MIDI, DJ-mix set-DNA, or authored knowledge
  (35 years of DJing). Inert data; does nothing on its own. Lives in a selecta's `profile:{}` (plus
  `corpus:[]`, the provenance list of which reference tracks conditioned it).
- **Yo Conditioning — the VERB.** The act of USING that DNA to bias the generator's parameters. This is
  "corpus conditioning" — the actual ML term for biasing a generative system on extracted features —
  reworded as a ONEMANSHYO pun without losing technical accuracy.

**It is not training and not retrieval.** No weights are learned; nothing is looked up at runtime; the
output is never stored. The DNA is extracted offline, committed, and then *biases a stochastic process*.
It tilts the dice; it does not write the result. Same seed reproduces a loop; a new seed rolls a fresh one
within the DNA's biases.

**Two roles of DNA:**
- **COMPOSITION DNA — what the music IS.** Mostly AMU-on-tracks. Fields: `mode`, `swing`, per-band
  densities (`drumDensity` / `bassDensity` / `melodicDensity`), `bassEntry`, `breakdowns`, `vocalRatio`,
  `structure`, `dynamics`, `bpm`.
- **MIXING DNA — what the DJ DOES** (the performance layer, imposed on top of the generated track).
  Authored by Wes. Under `profile.mixing`: `maxBpmJump`, `approach` / `approachBars` (tempo-approach
  before a hard-cut change — Chippy does not overlap/blend), and `moves:{}` (`kickCut`, `bassEqOut`,
  `leadDrop`, `reverbThrow` — each `{on, chance, minLen, maxLen}`, fired at musical boundaries with a
  probability so they're alive, not mechanical).

**Wired to audio (as of V1.8.0):**
- `mode` (app-wide minor lock) and `swing` (per-loop groove offset).
- **Density conditioning** — per-band densities bias each band's fire probability, normalized against the
  busiest band (drums), floored so a band never fully vanishes. Drum-forward JNO ⇒ drums full,
  bass/melodic thinned to their measured proportion. (`setDensityFromProfile` → `dGate` → `gk`, applied at
  the house builder's band-probability points.)
- **`maxBpmJump`** — the reroll clamps the tempo move between consecutive loops (JNO creeps ~2 BPM, never
  leaps). (`yoReroll` tempo block.)
- **Performance moves** — `buildMoves()` plans per-band suppression windows at 4-bar phrase boundaries,
  each gated by `chance`; `fireVoice()` honors them (cut drums/bass/lead, or force reverb on a throw).

**Captured but not yet wired:** `bassEntry`, `breakdowns` (needs a threshold definition), `vocalRatio` (no
vocal voice in the chiptune set), `structure`, `dynamics`, and mixing `approach` / `approachBars` (a true
tempo glide of the outgoing loop needs its own scheduler-clock iteration).

**The corpus** (JNO's committed Yo DNA source — 12 of Wes's own 124–128 house tracks, AMU'd via omsanalyze
v2): AintNoTrick, AllTheFun, AnotherBrickInTheWall, Breakaway (Goldilox), Curita, DiscoBarbie,
FrostyTheYoMan, HereComesTheSun (×2), JRSR400 DontTouch, JRSR541 FunkTrain, SevenNationArmada. Measured
means drove the wired values (drumDensity 0.654, bassDensity 0.447, melodicDensity 0.452, mode 12/12 minor,
~-9.8 LUFS, bpm 124–128). See §12 for the pipeline that produces this DNA.

*(Name note: "YoConditioner" is a working/beta placeholder for this system and may change.)*

---

## 2.5 Message System — the feedback bar (CORTEX)

Ported from Dojo's Message Bar (a system Dojo spent ~a month refining). It is the **single channel for
user feedback**, shown in a bar at the bottom of the app (`#messageBar`). Every module speaks through
one function; no module rolls its own feedback, and there are **no native `title=` tooltips** — all hover
help lives here.

**It is hover-driven, not action-driven.** The bar's primary job is to narrate *what you are pointing at
or doing right now* — mouse field to field and the bar tells you what each control is. Dojo's **two-phase
pattern**:

- **Phase 1 — ENTRY / hover (`info`, cyan `#00d4ff`):** on `mouseenter` over a control, show what it IS
  and its shortcut (e.g. "Tempo (BPM) — up/down pick, left/right scroll"). This replaces tooltips and is
  the constant, everyday use of the bar.
- **Phase 2 — CHANGE (`state`, orange `#ffa500`):** when you actually change a value, show the resulting
  value (e.g. "Tempo 124"). Confirmations of *values*, not a log of every action taken.
- **`error` (magenta `#ff006e`):** failures.

The bar is NOT an action log — "you clicked generate / muted a channel" style confirmations are not what
it's for. It reflects the control under the cursor (hover) and the value you just set (change).

- **`showMessage(text, type, onClick)`** — the one entry point; renders into `#messageBar`. `onClick`
  optional (clickable message). Types map to the three colors above.
- **`setMsg(text, color)`** — legacy shim retained during migration (cyan/none→info, else→state).
- **Central registry (`MSG`)** — THE single source of truth. Every user-facing string lives in one
  object: `MSG.hint` (phase-1 hover strings, keyed by control), `MSG.state` (phase-2 value-change
  messages as functions of the new value), `MSG.error` (failures). Nothing elsewhere holds a message
  string — to change any message, change `MSG`. Helpers `msgHint(key)` / `msgState(key,...vals)` /
  `msgError(key)` fire from the registry. Hover wiring: controls carry `data-hint="<key>"` (section
  labels) or are id/`data-*`-keyed; one boot-end pass attaches `mouseenter → info` for every entry.
  The full field/label table (C §12) is generated from `MSG` and mirrors it exactly.

**Why Cortex:** every part of the UI depends on it for feedback; changing it cascades to all user
communication; it enables future logging / i18n / message history.

---

# 3. NODE: THE PARTY GRID (automation constraint matrix)

The **Party Grid** is the internal constraint matrix governing ALL automation. It applies ONLY when a
selecta is active (any selecta other than **None**; the party button was retired in V1.9.0 — selecting a
selecta is now the mode switch, `yoOn`). Plain play (**None**) uses whatever's manually set. Three owners,
cleanly separated so they never fight:

- **SELECTA (`SELECTAS`)** owns **BARS + crate**: `barsAllowed` (allowed loop lengths, always real dropdown
  values — never multiplied off-grid), `barsChange` (chance/loop to re-pick a length), `pool` (which genres
  it draws from), `genreChance` (chance/loop to hop genre within the pool), and a bias `bpm` range.
- **GENRE (`GENRE[].tempo`)** owns **TEMPO**: each genre has a hard `[min,max]` BPM range (the outer clamp
  — never D&B at 75). The selecta's bpm range only biases *within* the genre range (effective = genre ∩
  selecta). Genres: house 120–126, techno 128–140, breaks 120–134, bigroom 128–132, dnb 150–190.
- **TIME SLOT (`VIBES`)** owns **LEVEL / DENSITY / PACE**: the club-night arc (Opener → Support → Headliner
  → Closer) scales master level (75/85/100/85), density, and loop pace (barBias).

A set opens at a bar length the selecta actually allows (not a hardcoded 128), snaps the genre into the
selecta's pool on start, and syncs the UI live as it evolves.

## 3.1 Selecta profiles (the Yo DNA carrier)
A selecta may carry an optional `profile:{}` — the repeatable Yo DNA schema (see §2.4), derived from AMU
analysis of real reference tracks (see §12). **Juice Night Out (`jno`)** is the first profiled selecta —
`{ mode:"minor", swing:[0,0.05], structure:[2,4,8], dynamics:"punchy", … }` — derived from 12 of Wes's own
124–128 BPM commercial house tracks (the JNO label catalog — the selecta IS the sound of the label). The
`none` selecta (index 0) carries `profile:null`: raw play, no conditioning, no auto-swap.

## 3.2 Set lifecycle
`startSet()` → snap genre into pool, set opening bars from `barsAllowed`, build + play the first
(conditioned) loop, sync UI. `yoReroll()` → stages the next loop (`pendingYo`) partway through the current
one; the scheduler applies it at the appropriate boundary via `applyPendingSwap`. Transitions are hard
cuts on the boundary/bar line (no crossfade concept yet — a documented future refinement). While playing, a
selecta change stages via `yoReroll` + `skipArmed` and swaps at the next bar line; while stopped it opens
immediately. `stopSet()` (selecta → None) stops the set.

---

# 4. NODE: VOICE / MODULE SYSTEM

## 4.1 VOICES data
`VOICES[]` — the 8 voices, each `{ id, name, color, osc, on, lvl, decay, … }`: drums, perc, bass, lead,
synth, accents, fills, **fx**. `V{}` is the id→voice lookup. The DRUMS lane is special (dedicated kick
synth); voice 8 (**fx**) is the effects voice.

## 4.2 Module UI (`buildStrip`)
Each voice renders as a `.devmod` box with two rows:
- **Row 1** — the voice-type **selector** (dropdown; `OSC_OPTS`, or `KICK_OPTS` for drums, or `FX_OPTS` for
  fx) + a **level** value (scroll/drag wheel via `bindWheel`).
- **Row 2** — three buttons: **random** (dice — re-rolls the voice type), **reverb** (concentric rings —
  per-voice reverb send, `v.fxRev`), **kill** (power — mutes the lane, solid red when engaged).

## 4.3 FX voice (voice 8)
`FX_OPTS`: **Riser Up / Riser Down** (smooth resonant low-pass sweeps), **Glitch Up / Glitch Down**
(chopped gated stutter — the "d-d-d" via `playGlitch`), **Impact** (short punchy stab), **Sweep** (broad
riser), **Ship Horn** (detuned-saw foghorn "BWAAAM" via `playHorn`, pitched to the track root). All
white-noise/synth generated; fires at phrase boundaries (risers into drops). Level is heavily attenuated
internally (FX is hot at full scale) and defaults dialed low. Per-voice reverb defaults **OFF** for the fx
voice (V1.9.03).

## 4.4 Master FX (VOICES header)
A master effects bus control in the VOICES header (right-aligned): a **toggle** (`masterFxOn`) + a
**strength** value (`masterFxAmt`, scroll/drag wheel). When on, every voice EXCEPT drums routes through the
reverb bus at the chosen strength (drums stay dry — no washy kicks). Defaults ON at ~10% for the house
signature. Keyboard-reachable via the WASD grid.

---

# 5. NODE: MATRIX / PIANO ROLL

The scrolling piano roll (`#tl` canvas). `LANEH` per-lane height; `ROLLH = RPAD*2 + LANEH*8` (fits 8
lanes); `RPAD` insets the lane block evenly from the matrix frame.
- **Lanes** — 8 voice lanes, notes drawn as colored blocks scrolling past a fixed playhead. The playhead
  and scroll read the **continuous** session clock (see §2.1.1), so they stay dead-center across loop
  boundaries.
- **Lane labels** — a modular master-wrapper box holding uniform-width label chips (DRUMS…FX), pinned to
  their lanes; shaded so scrolling notes stay readable behind them.
- **Overview strip + Chippy the conductor** — a framed strip above the roll. Chippy rides the **loop**
  clock (`masterLoopClock`), inset by his radius so his whole body stays inside the frame; he restarts at
  the left at each real loop boundary (including a selecta switch) and never vanishes early.
- **Session counter** (MATRIX header) — two independent boxed readouts: the **loop** clock
  (`bar . beat . six`, blinks on its last bar as a change heads-up) and the **session** clock
  (continuous Ableton-style `bars.beats.sixteenths`, resets to 1.1.1 on stop). Same two-view model as
  §2.1.1.

---

# 6. NODE: GLOVER (visualizer)

Full-screen beat-reactive light show on the GLOVER tab (`drawRave`). The Chippy family (Papa + colored
kids orbiting) is always present and unchanged; the **background** carries the trick. Named after "gloving"
(LED-glove light shows). Six trick modes (`gloverTrick`): **Tracers** (comets on lissajous paths with
bleeding trails), **Liquid** (undulating wave-bands), **Tutting** (LED grid + a line hopping in sharp 90°
angles), **Strobe** (radial sun-rays + random-flashing beams), **Orbits** (spirograph light-painting),
**Bloom** (Simon-style LED wall blooming in/out). Beat energy is read from `hit` onsets
(kick/snare/bass/lead/crash/bar) via `env()`. The Glover canvas records (MP4 where supported, else WebM);
audio tapped from master.

---

# 7. NODE: TITLE GENERATOR

Generated loop names via `makeTitle()`. Word banks are **classed by provenance** (a hard rule — only
Wes-contributed terms go in his classes; Claude-added connective words are quarantined and labeled):
- **BRANDS** (Wes) — retired '90s/Y2K rave brands (G.A.T., MacGear, Caffeine, Stuka, …).
- **MONIKERS** (Wes) — DJ heroes, possessive (Coxy's, Fatboy's, Slim's, Sven's, Paul's).
- **VENUES** (Wes) — venues (Simon's).
- **FX_DESC / FX_END** (auto flavor, labeled) — connective descriptors/endings.
Combined into varied patterns ("Coxy's Warehouse", "Live at Simon's", "Phat Caffeine 47"). Fires on every
generate and set swap; the INFO genre field + `patName` update alongside.

---

# 8. NODE: TAB / SECTION STRUCTURE

Three tabs (leaf nodes of the top-level UI), switched via a `.panel.active` class; persistence (∞) keeps
audio alive across tab switches:
- **MUSIC** (`#genTab`/`#panelGen`; internal `data-panel="gen"`) — the main instrument. Renamed from
  TRACKS (V1.9.16); internal ids kept.
- **GLOVER** — the full-screen beat-reactive light show (§6).
- **ABOUT** — credits, philosophy, technical notes, PLUR.

**MUSIC-tab areas — canonical names + code handles** (every area is named for docs/reference even when
it shows no UI header):

| Area | UI header? | Code handle | What it is |
|---|---|---|---|
| Summary row | — | `#summaryRow` | the top row holding INFO / MONITOR / FACE |
| **INFO** | yes | `#infoBox` | static track description: name, genre, source/seed, key, length |
| **MONITOR** | no | `#monitorBox` | live performance readout: master clock, loop clock, tempo |
| **FACE** | no | `#faceBox` | Chippy's reactive face canvas |
| **TRANSPORT** | yes | `.ctrl-col-transport` | play/pause + generate |
| **SELECTA** | yes | `.ctrl-col-selecta` | name · genre · time slot · bars · tempo · swing · q |
| **VOICES** | yes | `#devStrip` (+ SOURCE readout) | 8 voice-type selectors + dice |
| **MIXER** | yes | `#mixStrip` (+ AUDIO EFFECTS / MUTE) | per-channel level/solo/kill; master FX bus |
| **MATRIX** | yes | `#chartWrap` | overview strip (Chippy rider) + scrolling piano roll |
| **message bar** | — | `#messageBar` | the Message System's feedback bar (§2.5) |

**INFO vs MONITOR** are a pair: INFO = *static* (what the track is), MONITOR = *live* (what it's doing now
— master/loop/tempo, using INFO's grid grammar for visual consistency). The **SOURCE** readout in the
VOICES header (`#sourceBox`) names the hardware the synthesis is modeled after — currently **Ricoh 2A03**
(§2.2); "source" (not "chip") so it stays correct for a future non-chip model. TRANSPORT and SELECTA are
two peer sections sharing one header row (V1.9.11).

---

# 9. LEAF REFERENCE (controls + keyboard)

**Transport** (Controls → Transport module): play/pause · randomize (⚄ generate) · kill (master mute —
silences the mix without stopping transport). (The old ⏭ next button was removed — redundant with
generate, since there's no playlist.)

**Keyboard / WASD grid navigation:** WASD navigates the control grid (rows: Controls, Voice selectors,
Voice randoms, Voice kills, plus Master FX). A/D move within a row (wrapping across rows as one ring); W/S
jump between rows (land at the left). Arrow keys change the focused control's value. Tab is untouched.
Space = play, G / N = generate.

**Other leaves:** persistence (∞), orientation (16:9 / 9:16 for Glover recording), record, per-voice
reverb, master FX strength, quantize readout (Q ♩), selecta/time-slot selectors.

---

# 10. THE DESIGN SYSTEM (governed — NOT Cortex)

The universal UI/UX rules: tokens, typography, layout/spacing, color semantics, component patterns, and
anti-drift hard rules. It **governs visual consistency** across every section and tab — new UI inherits its
tokens/classes rather than hand-rolling a one-off. It is **important and governed, but it is NOT Cortex**:
breaking it makes the app look like amateur slop, not stop working. (It earned a written spec after real UI
drift — a magenta header, an oversized master-FX cluster, invented labels — proved the tokens needed to be
documented law, not just values loose in the CSS.)

The **full token values, color table, component classes, and the six hard rules** live in the Y System
Reference (§ "The Design System") so they sit next to the other machine-readable identifiers. This section
records *what the Design System is and why it's governed*; Y records *the exact values*. Core rules in
brief: section headers are always cyan; new UI inherits `.sec-hdr` / `.devmod` / `.dm-btn`; module controls
are a uniform 30 px height; no text labels on modules except the Matrix lane chips.

Lineage: Chippy inherits the OMS design language from the Sozo/Dojo template (dark rack aesthetic,
Eurorack-style module boxes, cyan section headers) — a scaled-down sibling of the Dojo Design System, same
language, Chippy's own values.

---

# 11. THE AMU → PROFILE PIPELINE (upstream: how Yo DNA is produced)

**The concept has a name: YO DNA (the artifact) and YO CONDITIONING (the method)** — see §2.4 for how
Chippy *uses* it; this section is how it's *produced* upstream.

The lineage: **Dojo** (first, ~2025) established the OMS single-file / module / audio-engine framework and
vocabulary. **Sozo** is the analysis app — it wraps Apple's **Music Understanding** framework via the
**`omsanalyze`** Swift CLI (`MusicUnderstandingSession(asset:).analyze()`), emitting a JSON sidecar per
track: exact integer BPM, beats/bars as absolute timestamps, key as time-ranges, loudness (LUFS/peak), and
song **structure** (sections/phrases/segments). Sozo also has an HTML visualizer for reading that JSON (its
squares/lines visual grammar is why Chippy's matrix looks as it does).

**Workflow:** run `omsanalyze` on reference tracks (e.g. Wes's own catalog) → get JSON sidecars named to
match each track → analyze *across* the set for the common signature (mode, swing/groove, BPM range,
structural bar-grid, loudness/dynamics) → encode that signature as a selecta `profile:{}` (§3.1). This is
not sampling or playback — it learns the *DNA* (fingerprint) of a style and teaches the generator to
produce fresh patterns that carry it. JNO (from 12 of Wes's 124–128 house tracks) is the proof-of-concept.

**omsanalyze v2** now emits real `instrumentActivity` (bass/drum/other/vocal, each with per-time activity
curves + ranges), previously stubbed/TODO — so per-band DENSITY is AMU-derived and wired (§2.4). Per-drum
onset PLACEMENT (exact kick step patterns) is still not extracted — that's the micro layer MIDI DNA will
supply. So profiles currently shape groove / key / structure / dynamics AND per-band density, not yet exact
kick patterns.

---

# 12. MESSAGE REGISTRY — EVERY FIELD & LABEL

The complete table of every user-facing message, generated from the central `MSG` registry (§2.5) and
mirroring it exactly. To change any message: edit `MSG` in the inline script, then regenerate this table.
This is the one place to look up or change what any control says.

### Phase 1 — hover hints (`info`, cyan)

| Key | Hint text |
|---|---|
| `lblInfo` | INFO — the current loop: name, genre, source, key, length |
| `lblTransport` | TRANSPORT — play/pause and generate a new roll |
| `lblSelecta` | SELECTA — the set: name, genre, time slot, bars, tempo, swing, quantize |
| `lblVoices` | VOICES — pick each lane's sound; SOURCE shows the modeled chip |
| `lblMixer` | MIXER — per-channel level, solo, kill; master audio effects + mute |
| `lblMatrix` | MATRIX — the live piano roll and whole-loop overview with Chippy |
| `genTab` | MUSIC — make loops: pick genre + length, generate, or run a selecta set |
| `raveTab` | GLOVER — full-screen beat-reactive light show |
| `aboutTab` | ABOUT — what Chippy is, who made it, the license |
| `bgBtn` | Persistence — keep playing when you switch tabs |
| `aspectBtn` | Orientation — 16:9 (YouTube) / 9:16 (Shorts, TikTok, Reels) |
| `recBtn` | Record the Glover visuals — click again to stop & download |
| `playBtn` | Play / pause (Space) |
| `genBtn` | Generate a new roll (G) |
| `djSel` | Selecta — None = raw play; pick one to run a conditioned set |
| `genreSel` | Genre — House (others return once the House conditioner is proven) |
| `vibeSel` | Time slot — Opener / Support / Headliner / Closer (pace + level) |
| `lenSel` | Bars — loop length; longer loops build through more phrasing |
| `bpmSel` | Tempo (BPM) — up/down pick, left/right scroll live |
| `swingSel` | Swing % — up/down pick, left/right scroll live |
| `quantReadout` | Quantize — control changes apply on the next beat |
| `sourceBox` | Sound source — the hardware Chippy's synthesis is modeled after |
| `vhKillBtn` | Mute — silence the whole mix without stopping playback |
| `chartWrap` | MATRIX — live piano roll; top strip is the whole-loop overview with Chippy |
| `data-rand` | Randomize this module's voice |
| `data-sel` | Pick this voice's sound |
| `data-lvl` | Level — scroll or drag |
| `data-solo` | Solo — play only soloed channels |
| `data-mute` | Kill — mute this channel |
| `data-mfx` | Audio effects — reverb on the whole mix except drums |
| `data-mfxamt` | Audio Effects strength — scroll or drag |

### Phase 2 — value-change messages (`state`, orange)

| Key | Format |
|---|---|
| `tempo` | `v => "Tempo "+v` |
| `swing` | `v => "Swing "+v+"%"` |
| `bars` | `v => v+" bars"` |
| `genre` | `v => "Genre "+v` |
| `selecta` | `v => "Selecta "+v` |
| `vibe` | `v => "Time slot "+v` |
| `voice` | `(name,val) => name+" "+val` |
| `level` | `(name,val) => name+" "+val` |
| `mute` | `on => on?"Muted (still playing)":"Unmuted"` |
| `solo` | `(name,on) => name+(on?" solo on":" solo off")` |
| `kill` | `(name,on) => name+(on?" killed":" on")` |
| `masterFx` | `on => "Audio effects "+(on?"on":"off")` |
| `persistence` | `on => "Persistence "+(on?"on":"off")` |
| `orientation` | `v => v` |
| `glover` | `v => "Glover "+v` |
| `generated` | `g => "Generated "+g+" roll"` |

Errors (`error`, magenta): `recNoSupport` → "Recording not supported in this browser".

---

*Living document. Companion to the Y System Reference (identifiers). Mirrors the OMS Dojo Developer
Documentation framework (Cortex / Node / Leaf) at Chippy's scale. Grows as Chippy matures.*
