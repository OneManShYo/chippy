# OMS CHIPPY [Beta V1_14_0] — DEVELOPER DOCUMENTATION

**Run date:** 2026-09-03  
**Current Unix Epoch:** 1788473550  
**App Version:** Chippy Beta V1_14_0  
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
  via `reverbWet`. Every voice EXCEPT drums routes through it when the master **reverb** (part of the
  audio-effects rack, §4.4) is on. Drums are excluded by design (dry kicks). NOTE: the reverb is one of
  four master audio effects (reverb·echo·filter·backspin); the whole rack is named `audioEffects*`, never
  "fx" (which is the FX voice, a sound).
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
  probability so they're alive, not mechanical). **As of V1.11.0, `reverbThrow` — the one audio-EFFECT
  move — is NULLED** (removed from the `buildMoves` spec, `forceRev` path left dormant). Auto-applying
  audio effects is parked until the now-full effects rack is better understood; only the composition/
  arrangement moves (kickCut/bassEqOut/leadDrop) fire. Re-enable = restore the `["reverbThrow","__rev",…]`
  spec row (BLOG0066).

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

---

## 2.4bis STRUCTURE DNA — the fourth role (expanded, V1.11.x exploration)

The two roles above (Composition / Mixing) are joined by a THIRD authored role — **STRUCTURE DNA: what the
producer BUILT** (arrangement, form, energy arc, section grammar) — plus AMU's single **harmonic** field.
Four roles total:

| Role | Describes | Authored by | Source | Wired? |
|------|-----------|-------------|--------|--------|
| **COMPOSITION DNA** | what the music IS (mode, density, swing, bpm) | machine | AMU-on-tracks | partly (§2.4) |
| **STRUCTURE DNA** | what the producer BUILT (form, sections, energy arc) | hybrid (L0–L3 below) | commentary + template + session + AMU | NOT yet |
| **MIXING DNA** | what the DJ DOES (live performance moves) | Wes | authored | partly (§2.4) |
| *(harmonic anchor)* | key/mode | machine | AMU `key` (one field) | no |

STRUCTURE DNA draws on FOUR source-layers, ranked by AUTHORITY — and the ranking is INVERTED from how a
machine would weight it: **the producer is the top, the machine the bottom.**
- **L0 — PRODUCER COMMENTARY** (§2.4bis.0) — intent/reference/lineage. Never machine-capturable.
- **L1 — AUTHORED TEMPLATE** (§2.4bis.1) — the JNO marker grammar Wes + Paulie designed.
- **L2 — SESSION** (§2.4bis.2) — the .als ground truth: locators + clip map.
- **L3 — MEASURED** (§2.4bis.2) — AMU. Corroboration + density texture only. Never primary.

**Why the sources are kept in SEPARATE tables (design rationale).** Coverage VARIES per track and this is
expected: some projects are MIDI-rich (transplantable notes), some all-audio (AMU density only); some carry
deep producer commentary, some none; some have full session locators, some are works-in-progress. A track's
DNA is STRONGER in some areas, WEAKER in others. Separate per-source tables mean **each track contributes
its strengths without being forced into a uniform shape** — a MIDI-rich/comment-poor track and a
comment-rich/MIDI-poor track both contribute cleanly. Then you **correlate ACROSS** the tables where the
signal overlaps (e.g. the AintNoTrick sub playing E1 confirms L0 commentary + AMU key + AMU half-time + the
MIDI, all at once). Do NOT collapse the sources into one table: the whole point is that each is a different
kind of truth with different coverage, and the cross-correlation is where the fingerprint sharpens. It is,
as Wes put it, "mixing but with AI" — every source on its own channel, brought up or down by ear.

### 2.4bis.0 STRUCTURE DNA — Producer Commentary (L0, the top layer)

**The Song Exploder layer — the person who made the file.** Highest authority in all of Yo DNA. This is
Wes breaking down his own track the way an artist does on *Song Exploder* (Hrishikesh Hirway's show:
artists provide stems and narrate why each part exists). It is **intent, reference, and sound-design
lineage** — the *why* behind the notes. **No AI, no Apple Music Understanding, no analysis will ever
capture this**, because it is not in the audio: it is in the head of the person who built it and played it
a hundred times.

**Authority hierarchy (inverted from how a machine would rank it — the producer is the TOP, the machine
the BOTTOM):**
- **L0 — PRODUCER COMMENTARY** — intent/reference/lineage. Wes. Never machine-capturable. *(this section)*
- **L1 — AUTHORED TEMPLATE** — the JNO marker grammar Wes + Paulie designed. (§2.4bis.1)
- **L2 — SESSION** — the .als ground truth: locators + clip map. (§2.4bis.2)
- **L3 — MEASURED** — AMU. Corroboration + density texture only. Never primary. (§2.4bis.2)

**Capture philosophy: record everything, mix later.** Commentary is captured thorough and un-enumerated —
part structured (fields that can eventually *bias* Chippy: bass character, energy, references), part free
prose (the full story). Not every track fills every field; that's fine. The model is **"mixing, but with
AI"** (Wes) — get every signal onto the board, then bring layers up or down by EAR: keep what makes Chippy
sound like Wes, pull down what muddies it or breaks the app. You can't know which enriches vs. which breaks
until you hear it, so keep it all available and sort by ear.

#### AINT NO TRICK — Producer Commentary (Wes)

| Field | Commentary | → possible Chippy bias |
|-------|-----------|------------------------|
| **Energy** | Intense, hyped — high-energy throughout, not subtle. | overall density/pace high; time-slot Headliner-leaning |
| **Hook** | Vocal hook "ain't no trick my DJ can't do" — a **half-time vocal flip**. | corroborates the half-time feel (BASSLINE stem read 90 vs 128) |
| **Breakdown** | **Old-school house bells** in the breakdown — deliberate classic-house reference. | breakdown = melodic/bell-tone element, not just a filter sweep |
| **Bass** | A **rumble bass** — intentionally mimicking the techno rumble-bass technique (distorted, reverb-fed sustained sub that "rumbles"). "Came out fucking awesome." | BASS voice on JNO tracks should lean **sustained / driving / gritty**, NOT plucky |
| **Lineage** | Built on the JNO template Wes + Paulie developed after ~a year studying house. | ties the track to the L1 authored template |

**Why this layer is load-bearing for Chippy:** "rumble bass" is a *production intent* that translates
directly into a *generation instruction* — Chippy's BASS lane, when JNO-conditioned, should render sustained
and gritty because that is what the reference IS, and that is knowable ONLY from the producer. AMU can
measure the rumble bass's spectrum; it can never know it was chased on purpose. That gap — intent — is the
whole reason Yo DNA is *Yo* DNA and not generic corpus analysis.

*(Free-prose commentary, references to specific techniques like "rumble bass," and the story of how a
template or sound came about are all welcome here verbatim — enumeration is not required. Capture thorough;
mix later.)*
### 2.4bis.1 STRUCTURE DNA — The JNO Authored Template (L1)

**Origin: authored by Wes + Paulie, a deliberate house-arrangement template applied across the ~30-35-track
Juice Night Out catalog.** Durations vary per song by design ("same lengths would be boring") - the MARKER
SEQUENCE is the constant, the LENGTHS are stochastic. You do NOT need all 30 tracks to know the template:
one well-mapped session gives the marker vocabulary; **probability + swing supply the variation.**

**The full marker sequence** (from the AintNoTrick session, JNO-SR021 - real locators, exact bars):

*(Source: **Sozo Sidecar** parse of the `.als` project file — not a screenshot, not memory. Ableton stores
every locator with exact beat/bar/second position, and Sozo extracts them directly. REPEATABLE across the
whole catalog: point Sozo at any JNO `.als`, get the locator sequence + exact bars in seconds. So Sozo is
BOTH the L3 tool (AMU listeners on the WAV stems) AND the L2 tool (the `.als` locator/structure parse) —
tool, not interpreter: Sozo extracts, Wes + Chippy interpret. That is the automated STRUCTURE DNA (L2) pipeline — 30-35 sessions -> the full arrangement-grammar corpus, machine-extracted from source,
no manual mapping. AintNoTrick confirmed: 128 BPM, 14 locators, clean 8/16-bar phrase blocks, boundaries on
exact beats 0/64/128/192/224/256/320/384/416/448/512/576/640/704.)*

| # | Marker | Bar | Bars | Role | Keep for Chippy? |
|---|--------|-----|------|------|------------------|
| 1 | **INTRO** | 1 | 16 | DJ mix-in bed | drop (mixing infra) |
| 2 | **BASS IN** | 17 | 16 | bass enters, energy starts | keep |
| 3 | **HOOK** | 33 | 16 | first hook statement | keep |
| 4 | **HP** | 49 | 8 | high-pass filtered transition | keep (transition) |
| 5 | **BUILD** | 57 | 8 | build into the main | keep (transition) |
| 6 | **MAIN** | 65 | 16 | Section A - full kick-driven groove | keep |
| 7 | **LIFTER** | 81 | 16 | tension lift into the breakdown | keep |
| 8 | **STRIPPED** | 97 | 8 | breakdown pt.1 - elements strip OUT | keep |
| 9 | **ATMO** | 105 | 8 | breakdown pt.2 - atmospheric bed (the old-school house BELLS live here) | keep |
| 10 | **BUILD** | 113 | 16 | rebuild out of the breakdown | keep (transition) |
| 11 | **MAIN** | 129 | 16 | Section B - the peak drive | keep |
| 12 | **LIFTER** | 145 | 16 | second lift | keep |
| 13 | **HOOK OUT** | 161 | 16 | final hook | keep |
| 14 | **OUTTRO** | 177 | 16+ | DJ mix-out bed | drop (mixing infra) |

**The breakdown is a THREE-move anatomy, not a generic "drop":** LIFTER (lift tension) -> **STRIPPED**
(pull the elements out) -> **ATMO** (atmospheric bed - this is where the old-school house bells sit, per L0
commentary) -> BUILD (rebuild) -> MAIN (peak). That strip -> atmosphere -> build sequence IS the JNO
breakdown grammar. Corrected from an earlier guess of a single "DROP" marker - the real session has the
two-part STRIPPED/ATMO move.

**Phrase math:** sections are clean 16-bar (content: INTRO/BASS IN/HOOK/MAIN/LIFTER/HOOK OUT) and 8-bar
(transitions: HP/BUILD/STRIPPED/ATMO) blocks - exactly the disciplined structure of a studied template.
The 8-bar blocks are the *moves*; the 16-bar blocks are the *content*.

**Length variation = existing machinery.** The per-section bar count is rolled stochastically within a range
(a 16-bar section might land 8/16/32; the big STRIPPED/ATMO breakdown varies most). This reuses the exact
`chance`/`minLen`/`maxLen` + swing mechanism Chippy already applies to mixing moves - re-pointed at SECTION
lengths. One session gives the sequence; probability gives the variation; no need for all 30 tracks.

**How Chippy uses it (the payoff):** Chippy does not INFER house structure from analysis - it GENERATES
along this authored sequence. Walk BASS IN -> HOOK -> HP -> BUILD -> MAIN -> LIFTER -> STRIPPED -> ATMO ->
BUILD -> MAIN -> LIFTER -> HOOK OUT (INTRO/OUTTRO dropped - Chippy loops, it doesn't beatmatch), each
section's length rolled within range, each section's element activation biased by the session clip map. That
replaces flat loop-forever with generation in Wes's actual compositional language. (Directly wires the
currently-captured-but-unwired `structure` / `breakdowns` / `dynamics` fields.)

### 2.4bis.1b MIDI DNA — the literal notes (from the .als, via Sozo)

**The surprise layer, and the strongest for composition.** Sozo parses not just the `.als` locators (L2
structure) but the **MIDI clips** — the actual programmed note events per instrument: `pitch`, `beat`,
`dur`, `vel`. AMU has no note-level pitch listener, so we assumed note transplant was impossible. **It is
not — the MIDI has the notes directly.** This is BETTER than any audio analysis for composition: it is not
measured-from-audio, it IS what Wes programmed.

**AintNoTrick MIDI (8 MIDI tracks, machine-extracted):**

| MIDI track | What the notes ARE | Chippy piano-roll use |
|------------|--------------------|-----------------------|
| **Kick (M)** | notes on beats 0,1,2,3,4… vel 112 = **four-on-the-floor**, verbatim | DRUMS lane pattern — direct transplant |
| **Clap 1/2 (M)** | placed hits (beat 33, dur 0.25) | PERC/accent lane |
| **Open Hat (M)** | beat 96.5 (the **offbeat** .5) | HAT lane — the swung placement |
| **Intro Sub (M)** | 96 notes, pitch 28 = **E1**, walking to 31 = G1 | BASS lane — the actual bassline, on the root |
| **STRING** | pitch 88, dur 28.0 = sustained 28-beat pad | SYNTH/pad lane — held chord tone |
| **Ghost Kick (M)** | pitch 60, sparse | sub-kick texture |

**The four-layer cross-confirmation (why this proves the whole architecture):** the Intro Sub MIDI plays
**E1 — the root of E minor** — which independently confirms FOUR sources at once: L0 producer commentary
("rumble bass" = root-driven sustained sub) + L3 AMU (BASSLINE read half-time) + L3 AMU (key = E minor) +
the MIDI itself (literally E1). Producer intent, machine measurement, and the actual notes all agree. That
convergence is the architecture working as designed — every layer pointing at the same truth from a
different angle.

**MIDI DNA is directly transplantable into Chippy's piano roll** — quantize the note beats against the
128 grid and you have Wes's actual kick pattern, bassline, hat groove, per lane. Unlike everything measured
from audio, this needs no inference: it is the note events. (Note: not all instruments are MIDI — the audio
stems cover the rest; MIDI DNA is richest where Wes programmed rather than recorded.)

#### 2.4bis.1b-impl — MIDI DNA, IMPLEMENTED (V1.11.01–04)

The stub above is the concept; this is the shipped mechanism. Five lanes were extracted from the AintNoTrick
MIDI JSON, folded from absolute song-beats to within-bar figures, and wired into `house()` as
**optional-with-fallback** reads off the active selecta's profile.

**Home — `profile.midi.*` (source-nested).** MIDI DNA lives under `profile.midi`, kept SEPARATE from the
flat AMU-measured composition fields (`drumDensity`…) and from `profile.mixing`. This mirrors §2.4bis's
"keep the tables separate per source; correlate across." A pattern is a *sequence* (the actual hit
positions), not a scalar bias — so it wants its own namespace, not a seat next to a density float.

```
profile.midi = {
  kick: { pattern:[0,1,2,3],           vel:112 },              // beats — four-on-floor
  hats: { pattern:[0.5,1.5,2.5,3.5],   vel:92, rigid:true },   // beats — offbeat, dead-straight
  bass: { pattern:[0,3,6,8,11,14],     oct:-1, vel:94 },       // STEPS — E1 root rumble + 16th pushes
  clap: { pattern:[1,3],               vel:102 }               // beats — backbeat on 2 & 4
}
```

**Pattern unit is per-lane, by design.** kick/hats/clap store **beats** (their hits land on clean beat
divisions); bass stores **STEPS** (its 16th "a"-pushes on steps 3 & 11 don't fall on clean beats). The
grid mapping (`beat*4` → 16th step) lives in `house()`, so the DNA field stays in the source's own unit and
needn't know Chippy's 16-step grid. This is a deliberate per-lane choice, not a schema inconsistency.

**Read path — optional-with-fallback (the coverage-varies principle).** Each lane read is
`selecta()?.profile?.midi?.<lane>`. Present ⇒ seed the lane from it. Absent (None selecta, or a MIDI-poor
track) ⇒ the original stock generation runs unchanged. The code MUST work whether a profile has `midi` or
not — some tracks are MIDI-rich, some aren't.

**The audible-vs-plumbing distinction (a real finding).** Chippy's stock house skeleton ALREADY equals
AintNoTrick's drum grammar, so two of the four lanes are position-identical to stock:

| Lane (iter) | Folded pattern | vs stock generation | Audible change | Character |
|---|---|---|---|---|
| kick (V1.11.01) | `[0,1,2,3]` beats → steps 0,4,8,12 | **identical** to stock four-on-floor | none | **plumbing proof** — identical output isolates "does the read/seed/fallback path work" from "does it sound different." Verified by generated-array inspection, not ear. |
| hats (V1.11.02) | `[.5,1.5,2.5,3.5]` → steps 2,6,10,14 | same POSITIONS, but stock adds a stochastic ghost (step 7) | **yes — feel** | Wes's hat is rigid/no-ghost; `rigid:true` suppresses the stock ghost roll. The signature is machine-tightness, not a new position. |
| bass (V1.11.03) | `[0,3,6,8,11,14]` steps, deg 0 oct −1 = E1 | stock is offbeat walk `[2,6,10,14]` on degrees | **yes — position** | Root-driven E1 rumble, on-beat + 16th pushes. Monophonic root 84% (a rumble, not a melodic line). The lane where Wes's signature is loudest. |
| clap (V1.11.04) | `[1,3]` beats → steps 4,12 | **identical** to stock snare backbeat | none | **plumbing-class** (like kick). Clap 1 == Clap 2 (layered for thickness, one pattern). Only real diff: voice identity (`clap:true` vs stock `snare`). |

**Design consequence for the docs/roadmap:** the MIDI DNA's audible contribution is concentrated in
**feel** (hat rigidity) and **the sub bassline** (genuinely different positions) — the drum skeleton merely
*confirms* what stock already generates. This is not a shortfall; it's the DNA telling us Chippy's
music-theory house engine was already right about the drum grammar.

**Parked: FILL DNA (not yet a lane).** Off-grid / section-boundary events do NOT belong in the base loop
pattern and are deliberately held for a future section-boundary iteration: (1) the second Kick track carries
**54 off-grid 16th-note kicks** (steps 1,2,3,5,6,7,9,10,11,13,14,15) clustered at bar turnarounds — kick
double-ups/rolls, not base pattern; (2) the sub-bass **melodic turnarounds** (E1→D2→A1→G1 descending lick)
at section boundaries; (3) the **STRING** one-shot (a single held note, a section-entry marker, not a loop
figure). All three are *section-event DNA*, tied to arrangement position — they hook onto the section engine
(§2.4bis.3), not the per-bar generator.

### 2.4bis.2 STRUCTURE DNA — Ain't No Trick (session instance, L2 + L3)

**Source: JNO-SR021 session (.als locators + clip map) — ground truth.** 181 bars, 6:03, 128 BPM, E minor.

**Section spine (L2 session locators) + energy corroboration (L3 AMU `pace`):**

| Session marker (bar) | AMU pace (energy) | Notes |
|----------------------|-------------------|-------|
| INTRO (1) | 16 (low) | DJ mix-in |
| BASS IN (17) / HOOK (33) | 32 (high) | drive begins |
| HP (~49) | dip | filtered mini-break |
| MAIN (65) | 32 (high) | Section A |
| LIFTER (81) | 32→ | tension lift |
| STRIPPED (97) → ATMO (105) | 16 (low) | **the big breakdown** — strip out, then the atmospheric bell bed — widest waveform dip |
| BUILD (113) | 21 (rebuild) | rollout |
| MAIN (129) | 32 (high) | Section B — peak drive |
| LIFTER (145) / HOOK OUT (161) | 32 (high, sustained) | sustained peak |
| OUTTRO (177) | 16 (low) | DJ mix-out |

*Three independent sources agree — session locators, AMU pace, and the raw waveform envelope all show:
low intro → drive → mini-break → drive → BIG breakdown → rollout → sustained peak → low outro.*

**Per-lane density fingerprint (L3 — AMU per-stem bpm vs true 128):** Kick_M 128 (four-floor) · Clap 128 ·
Hat_Loop 128 (16ths) · BASSLINE 90 (half-time feel) · GHOST_KICK 40 (sparse) · 70s_Disco_Riff 90.

**Element activation by section (L2 — from the session clip map):** e.g. DROP BASS enters only at the
DROP/breakdown; the 4 VOX phrases (WATCH OUT / MAKE YA GET DOWN / BRING THAT BEAT / AINT NO) land at hooks;
FX risers/reverses cluster at every section boundary (transition tools). *(Full per-section activation
matrix TBD as more sessions are mapped.)*

### 2.4bis.3 THE SECTION ENGINE — arrangement DNA → song shape (V1.11.05)

The lane transplants (§2.4bis.1b-impl) decide *what plays inside a bar*. The section engine decides *the
shape of the whole song* — where the intro sits, where the kick drops for a breakdown, how long the drop
runs. It is the fifth DNA input, and it comes with two hard rulings that govern the entire Yo DNA system.

**The arrangement is per-song DATA; the formula is the constant.** The `arrangement` block on a profile is
THIS song's section map — derived from the .als clip spans (which tracks enter/exit each 8-bar phrase). A
different song drops in its OWN `arrangement`. There is NO global template: if every song came out shaped
like AintNoTrick, that's one song wearing twelve costumes. The **formula** (`planSections()`) is what's
reusable; the **map** it reads is per-song.

```
profile.midi.arrangement = {
  contPlay: 0.25,                                  // continuous-play factor (see below)
  sections: [
    { role:'dj_util', kind:'intro',     bar:0,   len:8  },
    { role:'floor',   kind:'build',     bar:8,   len:8  },
    { role:'floor',   kind:'verse',     bar:16,  len:32 },
    { role:'floor',   kind:'breakdown', bar:48,  len:16 },
    { role:'floor',   kind:'verse',     bar:64,  len:64 },
    { role:'floor',   kind:'drop',      bar:128, len:32 },
    { role:'dj_util', kind:'outro',     bar:160, len:32 }
  ]
}
```

**THE FORMULA — two independent stages (`planSections(arrangement, BARS)`):**

- **Stage 1 — CONTINUOUS-PLAY FACTOR (fixed, structural, applied first).** Intro and outro (role
  `dj_util`) are cut to `contPlay` (~25%) of their real length, *regardless of budget*. Reason: a track's
  intro/outro are long only to give a **mixing DJ headroom** to blend over. Chippy plays **continuously —
  it never mixes** — so that headroom is dead air. A 32-bar outro is *inherently* ~8 bars in Chippy because
  24 of those bars only existed as mix runway. This is a correction for the non-mixing context, not a
  compression-to-fit — it happens before any budget math, independent of song length.

- **Stage 2 — BUDGET SQUEEZE (proportional, applied to the remainder).** After intro/outro are
  right-sized, the **floor sections** (build/verse/breakdown/drop — the song body) compress
  *proportionally* to fit whatever's left of the `BARS` loop-length budget. `BARS` is Chippy's existing
  loop-length setting (the `lenSel` dropdown) — re-purposed as the budget, no new control.

```
util'  = Σ dj_util.len × contPlay          // Stage 1: fixed trim
floorBudget = BARS − util'
scale  = floorBudget / Σ floor.len          // Stage 2: proportional
each floor.len' = round(floor.len × scale)  // (rounding drift reconciled onto the largest floor section)
```

Verified output (AintNoTrick's 192 real bars into three budgets):

| Budget | intro | build | verse | breakdown | verse | drop | outro | total |
|--------|-------|-------|-------|-----------|-------|------|-------|-------|
| 32  | 2 | 1 | 5  | 2  | 9  | 5  | 8 | 32  |
| 64  | 2 | 3 | 11 | 6  | 23 | 11 | 8 | 64  |
| 128 | 2 | 6 | 25 | 12 | 50 | 25 | 8 | 128 |

Intro (real 8 → 2) and outro (real 32 → 8) hold at the fixed 25% across every budget; the floor sections
breathe more as the budget grows; every plan lands exactly on the budget. At a bigger budget the drop gets
25 bars and the main verse 50 — the body has room; the intro still just *states itself* and moves on.

**RULING 1 — DNA is INFLUENCE, never a TEMPLATE.** The section plan BIASES the still-running generator; it
does not script it. Collapsing a generative engine into one memorized song is *lazy* and defeats the point
— you'd have a one-song sample library that supersedes the music-theory seed gen, and everything would
sound identical. So the section engine is the **lightest possible touch**:
- `yoArrange(P, plan)` thins probabilistically (`R() < mul`), it does NOT hard-drop voices. (The older
  manual-mode `arrange()` DOES hard-drop — deliberately NOT reused for Yo mode; too rigid by this ruling.)
- `sectionBias(kind)` returns per-section MULTIPLIERS on the existing `gk()`/`dGate()` weights — a
  breakdown *lowers* kick probability toward (but never to) zero; a drop *raises* density. The dice still
  roll; they just roll loaded.
- **Same song, two generations = same SHAPE, different DETAIL.** The DNA is the skeleton; the seed gen is
  still the flesh.

**RULING 2 — the section engine is ONE of several inputs.** It sits alongside the composition lanes, mixing
moves, density arc, fills, and key/mode. No single DNA input dominates. If the section engine ever starts
making everything sound the same, it is overweighted and wrong. It decides song *shape*; the other DNA
decides what plays *inside* each section; the generator decides the *actual notes*.

**Wiring:** Yo mode only, `BARS>=16` (sections need room to be meaningful). Manual mode keeps the old
`arrange()`. If the active selecta has no `arrangement` block, no section shaping runs — the per-bar
generator + lane DNA carry it, exactly as before.


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
  labels) or are id/`data-*`-keyed; `wireAllHints()` attaches BOTH `mouseenter` AND `focus` for every
  entry, so **keyboard/WASD/controller navigation lights the bar exactly like mouse hover** (BLOG0053) —
  Chippy is keyboard-first. `wireAllHints()` is idempotent and re-runs after `buildStrip`/`buildMixer`
  rebuilds so freshly-built strip controls keep both hover + focus feedback. A `var _hintsReady` gate
  prevents it touching `MSG` before `MSG` is initialized at boot (TDZ safety — see the boot-check note).
  The full field/label table (C §12) is generated from `MSG` and mirrors it exactly.

> **BOOT-CHECK DISCIPLINE (adopted this line).** `node --check` catches syntax only, NOT runtime
> temporal-dead-zone (TDZ) `ReferenceError`s (a `const` accessed before init at boot). Those halt boot
> partway → blank strips / empty MATRIX. Every UI-halting change is now **jsdom boot-checked** before
> ship: load the single HTML with `runScripts:"dangerously"`, filter canvas/`getContext` noise from
> `jsdomError`, and assert `bpmSel.options.length>0`, `devStrip.children.length===8`,
> `mixStrip.children.length===8`. (BLOG0039 / BLOG0053 / BLOG0059.)

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

## 4.2 Module UI (`buildStrip` / `buildMixer`)
The voices render in TWO strips:
- **VOICES strip** (`buildStrip` → `#devStrip`) — each voice a `.devmod` box: the voice-type **selector**
  (dropdown; `OSC_OPTS`, or `KICK_OPTS` for drums, or `FX_OPTS` for fx) + a **dice** button (`data-rand`,
  re-rolls the voice type).
- **MIXER strip** (`buildMixer` → `#mixStrip`) — each channel a `.dm-row` of three controls, left→right:
  **level** (scroll/drag wheel via `bindWheel`, `data-lvl`), **solo** (`data-solo`, yellow when engaged),
  **kill** (power, `data-mute`, solid red when engaged). NOTE: the per-channel **reverb** send was replaced
  by **SOLO** (BLOG0032); the additive solo model + the master-solo release live here (see §4.5).

## 4.3 FX voice (voice 8)
`FX_OPTS`: **Riser Up / Riser Down** (smooth resonant low-pass sweeps), **Glitch Up / Glitch Down**
(chopped gated stutter — the "d-d-d" via `playGlitch`), **Impact** (short punchy stab), **Sweep** (broad
riser), **Ship Horn** (detuned-saw foghorn "BWAAAM" via `playHorn`, pitched to the track root). All
white-noise/synth generated; fires at phrase boundaries (risers into drops). Level is heavily attenuated
internally (FX is hot at full scale) and defaults dialed low. The fx voice **defaults to Glitch Down**
(`osc:"glitchDn"`, BLOG0052). **FX is a SOUND (a voice), not an effect** — do not confuse it with the
audio-effects processing in §4.4 (which is deliberately named `audioEffects*`, never "fx").
NOTE: a vestigial **"Sweep FX"** entry was removed from `OSC_OPTS` (the tonal-voice source list) — it made
no sense on melodic voices and duplicated the real FX-voice Sweep (BLOG0051).

## 4.4 Master AUDIO EFFECTS (MIXER header) — the effects rack
A master audio-effects rack in the MIXER header (right-aligned), **left→right: reverb · echo · filter ·
backspin**, each button + wheel mirroring the last, no text labels (message-bar only). The whole family is
named **`audioEffects*` / `reverb*` — NEVER "fx"** (BLOG0055): "fx" belongs to the FX voice (a sound); this
is processing. Master output chain: `master → filterHP → filterLP → brakePitch → brakeLowpass → brakeGain →
out`.

- **Reverb** (`audioEffectsReverbOn` / `audioEffectsReverbAmt`, `data-audioeffects-reverb[-amt]`) — a
  send: every voice EXCEPT drums routes through the convolver reverb bus at the chosen strength (drums stay
  dry). Defaults **ON at ~10%** for the house signature. Button toggles, wheel = strength.
- **Echo** (`audioEffectsEchoOn` / `audioEffectsEchoAmt`, BLOG0056) — a master-bus `DelayNode` tapping the
  whole mix with a **300Hz high-pass in the feedback loop** (DJ low-cut echo — repeats don't muddy the low
  end). Wet returns in **parallel** to the output (never back into master → no runaway). **Tempo-synced
  3/16** (dotted-eighth), locked to BPM on play via `setEchoTime()`. Off by default, 30%. Wheel scales wet
  + feedback (feedback capped 0.85 so it always decays). `applyEcho()`.
- **Filter** (`audioEffectsFilterPos` / `audioEffectsFilterOn`, BLOG0057) — a bipolar single-knob DJ
  filter: two biquads (HP + LP) in series on the master bus. **Wheel: 50 = neutral** (both wide open,
  transparent); **<50 sweeps the lowpass DOWN** (muffle/underwater); **>50 sweeps the highpass UP**
  (thin/bass-out); log-frequency. **Button = bypass toggle** (keeps the wheel value — punch out to full,
  punch back to the exact sweep). `applyFilter()`.
- **Backspin / tape-stop** (`audioEffectsBrakeLen` / `audioEffectsBrakeHeld`, BLOG0058) — a **momentary
  HOLD** brake stage after the filter: `brakePitch` (a `DelayNode` whose delay time RISES on hold →
  Dopplers the mix DOWNWARD, the "vviuuup" spin-down), `brakeLowpass` (sweeps down), `brakeGain` (sags to
  silence). Press+hold engages (`brakeEngage`), release snaps back (`brakeRelease`). **The transport clock
  is untouched — only the audio is braked, so release resumes exactly on the live grid, no time lost.**
  Wheel = brake duration (~0.15s snap .. 1 bar sag, log, `brakeDurSec()`). Momentary via pointerdown/up.

All four are keyboard-reachable via the WASD grid (§9) and speak to the message bar on hover + focus.

## 4.5 Master SOLO + additive solo (MIXER header)
Per-channel solo is **additive** (click/click/click builds a solo group; touch-friendly, no modifier). The
**master solo** button (`vhSoloBtn`, BLOG0044) sits in the MIXER header: dim when nothing is soloed, lights
when ≥1 channel is soloed (via `_anySolo()`), and one tap **clears all solos** (returns the full mix). It's
a performance *release* gesture for subtractive mixing — solo elements down over a phrase, then slam the
whole mix back on the drop. The master **mute** (`vhKillBtn`) sits beside it.

## 4.6 Master channel processing (MIXER header) — EQ · comp · limiter · volume
The end of the master bus, after the audio-effects rack + master solo, before the kill switch (BLOG0061-64,
ported from the Dojo `V5_3_38d` DSP recipe). Each processor is a self-contained, labeled `.mproc` wrapper
(`data-mproc="eq|comp|limit|vol"`) — namespaced so any one can be retuned or later expanded into a slide-out
config panel (Dojo `dspPanelOpen` pattern) without touching the others. Labels via `.vh-mlabel` (same style
as the effect labels). **Naming is `master*`/`audioEffects*`, never "fx".** Full master output chain:

`master → [FX rack: filterHP/LP → brakePitch/LP/gain] → eqLow → eqMid → eqHigh → masterComp → masterLimit → masterVolGain → out`

- **EQ3** (`masterEqLow/Mid/High`, `applyMasterEq()`) — three biquads: `eqLow` lowshelf 250 / `eqMid` peaking
  1k / `eqHigh` highshelf 2500 (the "Ableton EQ3" recipe). Displayed **High·Mid·Low** left→right, natural
  **±12 dB** (0 = flat).
- **Compressor** (`masterCompOn`, `masterCompAmt`, `applyMasterComp()`) — one `DynamicsCompressorNode`.
  Button on/off; the macro wheel reports **ratio** (2:1 gentle → 8:1 hard) and sweeps threshold/ratio/knee
  across the gentle→aggressive presets. Bypassed (threshold 0, ratio 1) when off.
- **Limiter** (`masterLimitOn`, `applyMasterLimit()`) — a second `DynamicsCompressorNode` at the limiter
  preset (−3 dBFS ceiling, 20:1). Button on/off only. Dead-last in processing (catches the effects).
- **Volume** (`masterVolume`, `applyMasterVolume()`) — the final `masterVolGain` GainNode; a **live**
  control (not processing), at the chain end before kill.

All display values are on a normalized **0–10** scale (effects/comp/vol) mapped to the real DSP ranges;
EQ is the exception (natural dB). Everything defaults transparent, so the chain is inaudible until dialed.
Keyboard-reachable via the WASD grid, message bar on hover + focus. The **master solo** sits just left of
**VOL** (V1.10.24). Backspin engages on **pointerdown or keydown (Enter/Space)** and releases on up — so it
holds with a mouse, a finger, or a held key, and a future mapped controller uses the same path.

> **Web Audio note (compressor/limiter loudness):** `DynamicsCompressorNode` applies its own internal
> makeup — so turning COMP or LIMIT on makes the mix *louder/fatter*, not quieter. This is the node's
> default behavior, not an authored makeup-gain stage. For gain-matched processing later, add a
> compensating gain after each (or rebalance with master VOL). For a performance feel, "hit comp = it
> slams" is acceptable as-is.

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
| **INFO** | yes | `#infoBox` | static track description: name, genre, **seed**, key, length |
| **MONITOR** | no | `#monitorBox` | live performance readout: master clock, loop clock, tempo |
| **FACE** | no | `#faceBox` | Chippy's reactive face canvas |
| **TRANSPORT** | yes | `.ctrl-col-transport` | play/pause + generate |
| **SELECTA** | yes | `.ctrl-col-selecta` | name · genre · time slot · bars · tempo · swing · q |
| **VOICES** | yes | `#devStrip` (+ SOURCE readout) | 8 voice-type selectors + dice |
| **MIXER** | yes | `#mixStrip` (+ AUDIO EFFECTS rack / SOLO / MUTE) | per-channel level/solo/kill; master audio-effects rack (reverb·echo·filter·backspin) + master solo/mute |
| **MATRIX** | yes | `#chartWrap` | overview strip (Chippy rider) + scrolling piano roll |
| **message bar** | — | `#messageBar` | the Message System's feedback bar (§2.5) |

**INFO vs MONITOR** are a pair: INFO = *static* (what the track is), MONITOR = *live* (what it's doing now
— master/loop/tempo, using INFO's grid grammar for visual consistency). The **SOURCE** readout in the
VOICES header (`#sourceBox`) names the hardware the synthesis is modeled after — currently **Ricoh 2A03**
(§2.2); "source" (not "chip") so it stays correct for a future non-chip model. TRANSPORT and SELECTA are
two peer sections sharing one header row (V1.9.11).

---

# 8bis. NODE: THE LINEUP (session set-list scheduler) — V1.13.0

A **scheduler that sits ABOVE the conditioner** — it decides *which selecta is active when*, and never
touches generation, the YoConditioner grid, or the clock's audio path. Pure sequencing over time, driving
the existing selecta-swap machinery.

**Structure.** A styled right-anchored slide-out (`#lineupPanel`, injected in JS by `initLineup()` so it
never depends on markup parse order; toggled by `#lineupBtn` in the SELECTA row). Holds an ordered list of
slots — each slot is `{sel, min}` (a selecta index + a duration in minutes). Session-only, no persistence.

**Playback.** `Play Lineup` sets `luActive=true`, starts the transport if stopped, and engages slot 0.
`luEngageSlot(i)` converts minutes→bars — `luMinToBars(min) = round(min·BPM/4)` (min · BPM beats/min ÷ 4
beats/bar) — records `luSlotStartBar`, and engages the slot's selecta via `applyDj(sel)` (the existing
staged bar-boundary swap; no new machinery). `luTick()`, called each frame from `updateTime()`, counts
elapsed bars off `sessionClock().es / STEPS_PER_BAR`; when a slot's bar budget elapses it advances to the
next slot, ending after the last. **No new timing source** — it rides the master-loop clock and the existing
frame (CODEX0002 clean).

**Monitor integration (V1.13.0).** The MONITOR gained a "playing" now/next banner (`#nowNextCell` /
`nowNextScroll`, composed by `nowNextSummary()` / `updateNowNext()` each frame). `window._luNext()` exposes
the upcoming slot so the banner shows the next DJ the *whole* time the current slot runs (not just the
one-loop flash at handover). `window._luCountdown()` returns `{bar,beat,six}` remaining to the next swap;
`updateTime()` appends it to the loop row (relabeled **loop → music**) as `⇢ b . b . 16`, in the app's
bar.beat.16th standard. A pulsing pink running indicator (§10) lights `#lineupBtn` while `luActive`.

**Resident DJ + selecta-lock (V1.13.0).** `SELECTAS[2]` = **Resident DJ**, the pre-JNO "Giorgio Levan"
selecta (V1.5.0) restored verbatim — a scalar-only profile (`mode:minor, swing:[0,0.05], structure:[2,4,8],
dynamics:punchy`), run through the same fork points as JNO (`setDensityFromProfile`, `buildMoves`) with no
`midi`, proving the conditioner works from a corpus-free authored profile. It is the default selecta on load
(`selectaIdx=2`; boots stopped). While a selecta is active, `lockSelectaFields()` disables genre/bars/tempo/
swing (the reroll overrides them — no-op made visible; None re-enables); the WASD nav mover filters disabled
fields so keyboard nav skips them.

---

# 9. LEAF REFERENCE (controls + keyboard)

**Transport** (Controls → Transport module): play/pause · randomize (⚄ generate) · kill (master mute —
silences the mix without stopping transport). (The old ⏭ next button was removed — redundant with
generate, since there's no playlist.)

**Keyboard / WASD grid navigation:** WASD navigates the control grid, defined by ONE centralized,
self-documenting descriptor — `navRows()` is the single source of grid truth (BLOG0048), reaching **every**
editable field in visual order. Rows: **TRANSPORT+SELECTA** (play·generate·selecta·genre·time-slot·bars·
tempo·swing) · **VOICES** (interleaved select·dice per module, DOM order) · **MIXER master** (reverb·echo·
filter·backspin buttons+wheels, then EQ H/M/L · comp · limiter · **master solo** · volume) · **MIXER
channels** (interleaved level·solo·kill per channel). The master row is now collected in **DOM order** (via
one querySelectorAll including `#vhSoloBtn`/`#vhKillBtn`), so WASD follows the visual layout exactly no
matter how the row is rearranged (V1.10.24). A/D move within/across as one ring; W/S jump between rows (land at the left); arrow keys change the focused
control's value (untouched — separate handlers). Tab untouched. Space = play, G / N = generate.
Interleaved rows are collected by walking the module containers, so a new per-module control is picked up
automatically. Every field speaks to the message bar on focus (keyboard-first, §2.5).

**Other leaves:** persistence (∞), orientation (16:9 / 9:16 for Glover recording), record, the master
**audio-effects rack** (reverb / echo / filter / backspin — §4.4), **master solo** + mute (§4.5), quantize
readout (Q ♩), selecta/time-slot selectors. Value wheels take **horizontal touch-drag** (vertical is the
browser's scroll gesture; mouse keeps vertical drag — BLOG0046).

---

---

# 9bis. CHIP VOICE-MODEL FAMILIES — 2A03 / SID (V1.14.0)

Chippy synthesizes each voice from Web Audio primitives modeled on a chip. As of V1.14.0 the chip is
SWAPPABLE via the SOURCE dropdown (`#sourceBox`, a real select: `Ricoh 2A03` / `SID 6581`), driving `chipMode`.

**Ricoh 2A03 (default).** Pulse voices = `PeriodicWave` built from the Fourier series of the duty cycle
(band-limited, no aliasing). Bass = fixed `triangle`. Noise = a noise buffer through a bandpass. Clean, bright.

**SID 6581 (`chipMode==="sid"`).** The whole difference is one insertion: every voice routes through the SID's
signature multimode FILTER — a `BiquadFilter` (lowpass, cutoff ~1400Hz, Q~8) created once (`sidFilter`, feeds
master), and voices connect via `voiceOut()` which returns `sidFilter` in SID mode (else `curDest`). The
triangle-bass also swaps to `sawtooth` in SID mode. Result: clean beeper → fat, resonant, squelchy C64 synth.
INTENTIONAL AUTHENTIC QUIRK: the filter is fixed and does NOT track `generate()`'s per-song transpose, so some
songs land the lead out-of-tune — exactly how real SIDs behaved (kept, not a bug; see CODEX0005). Stage 2/3
(ring-mod, hard-sync, PWM, ADSR) are parked. `voiceOut()` is the one seam; grep `chipMode`, `sidFilter`,
`voiceOut`, `#sourceBox`.

---

# 9ter. MOBILE — POCKET RAVE (portrait surface) (V1.14.0)

In portrait (`@media (orientation:portrait)`), `#pocketRave` shows and the desktop `.app` is `display:none`
(NOT reflowed — dodges the old narrow-width black-screen; two independent surfaces, CSS picks one). All mobile
code is namespaced `pr-*` / `pocketRave` / `initPocketRave`. It REUSES the real engine through bridges:
`_playLineup`/`_stopLineup`/`_isPlaying` (Chippy-tap runs the default lineup), `_gloverOn` (full-screen Glover),
`_nowNext`/`_luCountdown` (monitor). Key pieces: animated Chippy canvas IS the play button (`#prPlay`/`prToggle`,
rolls a random `chipMode` per new song via `prChipRandom` read in `generate()`); a Tron-neon monitor (`#prInfo`
static, `#prMon` roll-over now/next — letterbox flip, no scroll — and `#prClocks` static loop-position ⇢
selecta-countdown); a dim vertical-roll background (`#prRollBg`/`prDrawBg`, reads live `pattern`+`sessionClock`);
MUSIC/GLOVER tabs (`prTab`) where GLOVER goes true full-screen (`#prGlover.pr-fs`, z-index 300, `body.glover-fs`
hides the jack-bay + rails, tap anywhere exits); a Moogerfooger enclosure (walnut `border-image` cheeks,
brushed-metal `::before/::after` rails, `#prSafeTop` recessed port-bay with a power switch + jack-nuts, sized to
`env(safe-area-inset-top)` + `viewport-fit=cover`). Screen wake-lock (`prWakeLockOn/Off`) holds the screen awake
while playing. Landscape/desktop = the full app, unchanged.

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

**Active-state glow pattern (V1.13.0).** The standard "this control is active / on / live" treatment is a
combination of *border-color + a ~10% background tint + a box-shadow glow*, optionally animated as a pulse.
Colors carry meaning: **cyan `#00d4ff`** = standard active/selected (`.glovbtn.active`, `.sel` focus);
**green `#00e08a`** = persistence on (`.bgtoggle.on`); **pink `#ff006e`** = running/live (the Lineup-running
ring on `#lineupBtn.lu-running`, animated via the `luRingPulse` keyframe). New "is-live/running" indicators
reuse the pink animated ring rather than inventing dots or new treatments — one idiom, three meanings by
color. (This is a design-system pattern, deliberately NOT Cortex — a CSS active-state is not a load-bearing
system.)

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
| `audioEffectsReverb` | `on => "Audio effects "+(on?"on":"off")` |
| `audioEffectsEcho` | `on => "Echo "+(on?"on":"off")` |
| `audioEffectsFilter` | `pos => pos===-1?"Filter off (bypass)":pos===50?"Filter neutral":pos<50?"Filter low-pass "+pos:"Filter high-pass "+pos` |
| `audioEffectsBackspin` | `held => held?"Backspin braking":"Backspin released"` |
| `masterEq` | `(band,v) => "EQ "+band+" "+(v>0?"+":"")+v+" dB"` |
| `masterCompRatio` | `r => "Compressor ratio "+r+":1"` |
| `masterComp` | `on => "Compressor "+(on?"on":"off")` |
| `masterLimit` | `on => "Limiter "+(on?"on":"off")` |
| `masterVol` | `v => "Master volume "+v` |
| `persistence` | `on => "Persistence "+(on?"on":"off")` |
| `orientation` | `v => v` |
| `glover` | `v => "Glover "+v` |
| `generated` | `g => "Generated "+g+" roll"` |

Errors (`error`, magenta): `recNoSupport` → "Recording not supported in this browser".

---

*Living document. Companion to the Y System Reference (identifiers). Mirrors the OMS Dojo Developer
Documentation framework (Cortex / Node / Leaf) at Chippy's scale. Grows as Chippy matures.*
