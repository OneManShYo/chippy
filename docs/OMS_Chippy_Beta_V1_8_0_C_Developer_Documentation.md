# OMS CHIPPY [Beta V1_8_0] — DEVELOPER DOCUMENTATION

**Run date:** 2026-08-31
**Unix Epoch:** 1788134400
**App Version:** Chippy Beta V1_8_0 (live at chippy.onemanshyo.com)
**License:** GPL-3.0
**Purpose:** The technical spec for OMS Chippy — architecture, systems, and internals. Modeled on
the OMS Dojo Developer Documentation and using shared OMS vocabulary (Cortex / Node / Leaf), sized
to Chippy's actual (younger, simpler) scope. A living doc that grows as Chippy matures.

---

## TABLE OF CONTENTS
- 1. ARCHITECTURE OVERVIEW
- 2. CORTEX SYSTEMS (core — don't casually touch)  [incl. 2.5 The YoConditioner]
- 3. NODE: THE PARTY GRID (automation constraint matrix + selecta profiles)
- 4. NODE: VOICE / MODULE SYSTEM
- 5. NODE: MATRIX / PIANO ROLL
- 6. NODE: GLOVER (visualizer)
- 7. NODE: TITLE GENERATOR
- 8. LEAF REFERENCE (features, controls, keyboard)
- 9. THE AMU → PROFILE PIPELINE (Sozo / omsanalyze integration)

---

## 1. ARCHITECTURE OVERVIEW

### Single-File Philosophy
Chippy is one self-contained HTML file — HTML + inline CSS + inline JS, zero external dependencies,
no build step, no network calls, no audio files. Everything (synthesis, rendering, state) runs in
the browser from that one file. This is an OMS-suite-wide principle (shared with Dojo/Sozo): the app
must open and run offline, on any device with a browser (MacBook, iPad, phone, Chromebook), forever,
with no rot. Any change that would add an external dependency (a font server, a framework, a sample
library) violates this and is rejected. All synthesis is generated in code (Web Audio), all fonts are
system/web-safe, all data is in-memory.

### Generation, Not Playback
Chippy is a *generator*, not a player. There is no audio recording, no sample library, no fixed song.
Every loop is produced on the fly from a seed + generation rules. Same seed → same loop; new seed →
a fresh variation. "Next" and "generate" are the same act (there is no playlist to advance through).
This is the soul of the app: infinite, seeded, rule-based music — "anyone can be a OneManShYo."

### Tab Architecture
Three tabs (leaf nodes of the top-level UI):
- **TRACKS** — the main instrument: INFO, CONTROLS, VOICES, MATRIX sections.
- **GLOVER** — the full-screen beat-reactive light show (renamed from RAVE ON), with six trick modes.
- **ABOUT** — credits, philosophy, technical notes, PLUR.
Tabs switch a `.panel.active` class. Persistence (∞) keeps audio alive across tab switches.

### Section Structure (TRACKS tab)
Each section is a `.sec-hdr` (colored bar + title + rule) followed by its content:
- **INFO** — generated loop name (title), genre, source/seed, key, length; + the Chippy face canvas.
- **CONTROLS** — four Eurorack-style module boxes: Transport, Generate, Feel, Party.
- **VOICES** — the 8 voice modules in a strip; MASTER FX lives in the VOICES header.
- **MATRIX** — the scrolling piano roll (8 lanes) + the session/bar counter in its header.

---

## 2. CORTEX SYSTEMS (core — don't casually touch)

The Cortex is the small set of foundational systems everything depends on. As in Dojo, these are
OMS-level and change rarely and carefully; feature work happens in the Nodes/Leaves above them.

### 2.1 Web Audio Engine
All sound is synthesized via the Web Audio API — no files.
- **`actx`** — the AudioContext. **`master`** — the master gain node → destination.
- **Oscillator voices** — variable-duty **pulse** waves via `PeriodicWave`, a **triangle** bass, and
  a **noise** channel from a buffer source (LFSR-style). Each voice = oscillator/noise + a short
  snappy envelope. The *timbre* (not the voice count) is what makes it chiptune.
- **Noise buffers** — white `noiseBuf`; pink/brown buffers also generated (used by some FX).
- **Kick synth** — the DRUMS lane is a dedicated pitch-dropping sine + click transient (`playKick`),
  with flavors (909/808/punch/click/noise).
- **FX synths** — `playSweep` (resonant low-pass noise sweep, the riser/glitch/impact/sweep FX),
  `playGlitch` (gated stutter), `playHorn` (detuned-saw foghorn). See §4 (FX voice).
- **Reverb bus** — a `ConvolverNode` (`reverbBus`) fed by a generated decaying-noise impulse; wet
  return via `reverbWet`. Voices route to it per-voice (the module reverb button) and/or via the
  Master FX bus (§3 / §4). Drums are excluded from the master reverb bus by design (dry kicks).
- **`curDest`** — per-fire output routing: a voice's play function connects to `curDest`, which is
  `master` (dry) or a tap that also feeds `reverbBus` (wet), set at the top of `fireVoice`.

### 2.2 Master Clock / Scheduler
Real-time audio scheduling locked to the AudioContext clock — this is the hardest, most critical
system in the app (browsers were not built for sample-accurate audio).
- **`STEPS_PER_BAR = 16`** (16th-note grid). `BARS` = loop length; `TOTAL_STEPS` derived.
- **`LOOKAHEAD = 0.25`s** — the scheduler places events up to this far ahead of `now` each tick, so
  background-tab throttling / GC stutters don't drop notes.
- **`loopStartCtx`** — AudioContext time the current loop iteration started. **`playOriginCtx`** —
  a stable anchor set once at play, used as the fixed grid for beat-quantize (so quantize targets
  can't recede past the horizon — this fixed the original quantize bug).
- **`scheduler()`** — the core loop: computes `horizon = now + LOOKAHEAD`, applies pending
  beat-quantize swaps at the next beat, applies pending party swaps + length changes at loop
  boundaries, and schedules each event via `fireVoice` at `loopStartCtx + ev.t`.
- **`nextBeatCtx()`** — first beat strictly after `now` (+ small guard), computed from
  `playOriginCtx` so it's always reachable within the horizon.
- **Persistence pump** (`bgPump`) — a silent node keeping the context alive when the tab is
  backgrounded (Chrome throttles rAF/timers otherwise). Toggled by the ∞ button.

### 2.3 Control-Behavior Model (three categories)
Every control that can change mid-play falls into exactly one category. This is a core rule —
documented in-code above the control wiring — and dictates whether a control is quantized, live, or
deferred:
- **RESTART** (genre, DJ/selecta) → routed through `quantizeKey()`: waits for the next beat (Q),
  cleanly restarts the loop. These are "new song" changes.
- **CONTINUE** (tempo, swing, per-voice selectors + levels) → applied LIVE, immediately, no restart
  (like turning a knob). Continuous parameters of the song already playing.
- **WAIT FOR LOOP** (length / bars) → deferred via `pendingLen`, applied at the loop boundary so the
  current phrase finishes before the new length takes effect.
When adding a new control, classify it into one of these three; that determines its wiring.

### 2.4 Theme Constants & the Design System
`CY #00d4ff` (cyan), `MAG #ff006e` (magenta), `YEL #ffcc00`, `GRN #00e08a`, `PUR #c77dff`,
`ORG #ffa500`. Voice lanes each carry a color; the palette is reused across matrix, modules, and the
Glover visualizer for a consistent identity.

These constants are the seed of a fuller **Design System** — a Cortex-tier UI/UX spec (tokens,
typography, layout/spacing, color semantics, component patterns, and anti-drift hard rules) documented
in the Y System Reference (§ THE DESIGN SYSTEM). It's Cortex because it governs visual consistency
app-wide; changing a token means auditing the whole UI. Core rules: section headers are always cyan;
new UI inherits existing classes (`.sec-hdr`/`.devmod`/`.dm-btn`) rather than hand-rolling; module
controls are a uniform 30px height; no text labels on modules except the Matrix lane chips. (The spec
was written after UI drift — a magenta header, oversized master-FX, invented labels — proved the tokens
needed to be a documented law, not just values in the CSS.)

### 2.5 The YoConditioner (the conditioned generative layer) — CORE

**This is how the music gets generated.** The YoConditioner is the Cortex-tier system that sits ABOVE
the audio layer (currently the chiptune sound set — swappable) and decides WHAT gets generated. It is
not the audio engine, not the clock — those merely voice whatever it produces. It is the generative
brain, plus the layer that biases that brain toward a real musical identity.

Two primitives, kept strictly distinct (this separation is the whole point — it's what makes the method
teachable and replicable):

- **Yo DNA — the NOUN.** The committed reference information: the extracted fingerprint of a style.
  Its *source* can be anything Wes officially commits — AMU analysis of tracks (omsanalyze), MIDI,
  DJ-mix set-DNA, or authored knowledge (his 35 years of DJing). Yo DNA is inert data; it does nothing
  on its own. It lives in a selecta's `profile:{}` (+ `corpus:[]`, the provenance list of which
  reference tracks conditioned it).
- **Yo Conditioning — the VERB.** The act of USING that DNA to bias the generator's parameters. This is
  "corpus conditioning" — the actual ML term for biasing a generative system on extracted features —
  reworded as a ONEMANSHYO pun without losing technical accuracy.

**It is not training and not retrieval.** No weights are learned; nothing is looked up at runtime; the
generated output is never stored. The DNA is extracted offline, committed, and then *biases a stochastic
generative process*. It tilts the dice — it does not write the result. Same seed reproduces a loop; a new
seed rolls a fresh one, within the DNA's biases.

**Two roles of DNA (two tables, two sources of truth):**
- **COMPOSITION DNA — what the music IS** (how a track is built). Mostly AMU-on-tracks. Fields: `mode`,
  `swing`, per-band densities (`drumDensity`/`bassDensity`/`melodicDensity`), `bassEntry`, `breakdowns`,
  `vocalRatio`, `structure`, `dynamics`, `bpm`.
- **MIXING DNA — what the DJ DOES** (the performance layer, imposed on top of the generated track).
  Authored by Wes. Fields under `profile.mixing`: `maxBpmJump`, `approach`/`approachBars` (tempo-approach
  before a hard-cut change — Chippy does not overlap/blend), and `moves:{}` (kickCut, bassEqOut, leadDrop,
  reverbThrow — each `{on,chance,minLen,maxLen}`, fired at musical boundaries with a probability so they're
  alive, not mechanical).

**What is WIRED to audio (as of V1.8.0):**
- `mode` (app-wide minor lock) and `swing` (per-loop groove offset) — the originals.
- **Density conditioning** — per-band densities bias each band's fire probability, normalized against the
  busiest band (drums), floored so a band never fully vanishes. Drum-forward JNO ⇒ drums full,
  bass/melodic thinned to their measured proportion. Code: `setDensityFromProfile()`, `dGate()`, `gk()`,
  called at the top of `buildPattern` and applied at the house builder's band probability points.
- **Mixing: `maxBpmJump`** — party reroll clamps the tempo move between consecutive loops (JNO creeps
  ~2 BPM, never leaps). Code: the tempo block in `partyReroll()`.
- **Mixing: performance moves** — `buildMoves()` plans per-band suppression windows at 4-bar phrase
  boundaries (party only), each gated by `chance`; `fireVoice()` honors them (cut drums/bass/lead, or
  boost reverb on a throw). Code: `buildMoves()`, `moveAt()`, `_moveWindows`, `fireVoice()`.

**Captured DNA not yet wired** (present in the profile, no conditioning consuming it yet): `bassEntry`,
`breakdowns` (needs a threshold definition), `vocalRatio` (no vocal voice in the chiptune set — target
undefined), `structure`, `dynamics`, and mixing `approach`/`approachBars` (a true tempo glide of the
outgoing loop needs its own scheduler-clock iteration).

**Scope note (V1.8.0):** the app is HOUSE-ONLY while the YoConditioner is dialed in on JNO — the other
genre builders (techno/breaks/dnb/bigroom) are retained in code but unlisted, and reintroduced rebuilt to
this standard once House is proven. Mirrors the single-selecta focus move of V1.6.0.

**The corpus** (JNO's committed Yo DNA source — 12 of Wes's own 124–128 house tracks, AMU'd via
omsanalyze v2): AintNoTrick, AllTheFun, AnotherBrickInTheWall, Breakaway (Goldilox), Curita, DiscoBarbie,
FrostyTheYoMan, HereComesTheSun (×2), JRSR400 DontTouch, JRSR541 FunkTrain, SevenNationArmada. Measured
means drove the wired values (drumDensity 0.654, bassDensity 0.447, melodicDensity 0.452, mode 12/12
minor, ~-9.8 LUFS, bpm 124–128). See §9 for the AMU → profile pipeline that produces this DNA.

*(Name note: "YoConditioner" is a working/beta placeholder for this system and may change.)*

---

## 3. NODE: THE PARTY GRID (automation constraint matrix)

The **Party Grid** is the internal constraint matrix governing ALL automation. It applies ONLY under
party mode (party = the automation button); plain play uses whatever's manually set. Three owners,
cleanly separated so they never fight:

- **SELECTA (`DJ_MODES`)** owns **BARS + crate**: `barsAllowed` (allowed loop lengths, always real
  dropdown values — never multiplied off-grid), `barsChange` (chance/loop to re-pick a length),
  `pool` (which genres it draws from), `genreChance` (chance/loop to hop genre within the pool),
  and a bias `bpm` range.
- **GENRE (`GENRE[].tempo`)** owns **TEMPO**: each genre has a hard `[min,max]` BPM range (the outer
  clamp — never D&B at 75). The selecta's bpm range only biases *within* the genre range
  (effective = genre ∩ selecta). Genres: house 120–126, techno 128–140, breaks 120–134, bigroom
  128–132, dnb 150–190.
- **TIME SLOT (`VIBES`)** owns **LEVEL / DENSITY / PACE**: the club-night arc (Opener → Support →
  Headliner → Closer) scales master level (75/85/100/85), density, and loop pace (barBias).

Party opens at a bar length the selecta actually allows (not a hardcoded 128), snaps the genre into
the selecta's pool on start (so e.g. Jungle opens on D&B, not house), and syncs the UI live as it
evolves.

### 3.1 SELECTA PROFILES (the "DNA" system) — V1.4.x
A selecta may carry an optional `profile:{}` — a repeatable "DNA" schema derived from **AMU analysis
of real reference tracks** (see §9). This keeps selecta character consistent instead of ad-hoc.
Schema:
```
profile: {
  mode      : "minor" | "major" | null,   // forced tonality
  swing     : [min, max],                 // allowed swing fraction (0 = straight)
  structure : [phrase, segment, section], // nested bar grid for where variation happens
  dynamics  : "punchy" | "smooth"         // accent emphasis
}
```
**Juice Night Out** is the first profiled selecta — `{ mode:"minor", swing:[0,0.05], structure:[2,4,8],
dynamics:"punchy" }` — derived from AMU'ing 12 of Wes's own 124–128 BPM commercial house tracks
(the Juice Night Out label catalog — the selecta IS the sound of the label): 100% minor, dead-straight groove (0–5% swing), nested 2/4/8-bar
structure, loud/punchy (~-9.5 LUFS). On party reroll, the profile's swing is applied within its range
(mode is already minor app-wide). NOTE: `structure` and `dynamics` are captured but not yet fully
driving the pattern generator — that's the next layer of work.

### 3.2 Party Loop Lifecycle
`startParty` → snap genre into pool, set opening bars from `barsAllowed`, build + play the first loop,
sync UI. `partyReroll` → stages the next loop (`pendingParty`) partway through the current one; the
scheduler applies it seamlessly at the loop boundary via `applyPendingSwap`. Transitions are hard cuts
on the boundary/beat (no crossfade concept yet — a documented future refinement).

---

## 4. NODE: VOICE / MODULE SYSTEM

### 4.1 VOICES data
`VOICES[]` — the 8 voices, each `{ id, name, color, osc, on, lvl, decay, ... }`:
drums, perc, bass, lead, synth, accents, fills, **fx**. `V{}` is the id→voice lookup. The DRUMS lane
is special (dedicated kick synth); voice 8 (**fx**) is the effects voice.

### 4.2 Module UI (`buildStrip`)
Each voice renders as a `.devmod` box with two rows:
- **Row 1** — the voice-type **selector** (dropdown; `OSC_OPTS`, or `KICK_OPTS` for drums, or
  `FX_OPTS` for fx) + a **level** value (scroll/drag wheel via `bindWheel`).
- **Row 2** — three buttons: **random** (dice icon — re-rolls the voice type), **reverb** (concentric
  rings — per-voice audio-effect send, `v.fxRev`), **kill** (power icon — mutes the lane, solid red
  when engaged).

### 4.3 FX voice (voice 8) — the FX types
`FX_OPTS`: **Riser Up / Riser Down** (smooth resonant low-pass sweeps), **Glitch Up / Glitch Down**
(chopped gated stutter — the "d-d-d" via `playGlitch`), **Impact** (short punchy stab), **Sweep**
(broad riser), **Ship Horn** (detuned-saw foghorn "BWAAAM" via `playHorn`, pitched to the track root).
All white-noise/synth generated. FX fires at phrase boundaries (risers into drops). Level is heavily
attenuated internally (FX is hot at full scale) and defaults dialed low; per-voice reverb defaults ON
for the fx voice (dialed-in house default).

### 4.4 MASTER FX (VOICES header)
A master effects bus control in the VOICES section header (right-aligned): a **toggle** (`masterFxOn`)
+ a **strength** value (`masterFxAmt`, scroll/drag wheel). When on, every voice EXCEPT drums routes
through the reverb bus at the chosen strength (drums stay dry — no washy kicks). Defaults ON at ~10%
for the house signature. Keyboard-reachable via the WASD grid.

---

## 5. NODE: MATRIX / PIANO ROLL

The scrolling piano roll (`#tl` canvas). `LANEH` per-lane height; `ROLLH = RPAD*2 + LANEH*8` (fits 8
lanes). `RPAD` = inner top/bottom padding (insets the whole lane block evenly from the matrix frame).
- **Lanes** — 8 voice lanes, notes drawn as colored blocks scrolling past a fixed playhead.
- **Lane labels** — drawn as a modular master-wrapper box (like the control modules) holding
  uniform-width label chips (DRUMS…FX), pinned to their lanes so they correlate with the notes; bold
  text; shaded so the scrolling notes stay readable behind them. (The colored "activity light"
  flicker on the left edge is notes passing under the translucent label chips — an emergent effect.)
- **Session counter** (MATRIX header) — loop-relative `bar · beat` + a running Ableton-style
  `bars.beats.sixteenths` session position (1-based, resets to 1.1.1 on stop), in a module box.

---

## 6. NODE: GLOVER (visualizer)

Full-screen beat-reactive light show on the GLOVER tab (`drawRave`). The Chippy family (Papa + colored
kids orbiting) is always present and unchanged; the **background** carries the trick. Named after
"gloving" (LED-glove light shows). Six trick modes selectable via buttons (`gloverTrick`):
- **Tracers** — comets streaking on lissajous paths with long bleeding trails (after-image).
- **Liquid** — flowing water wave-bands undulating across the canvas.
- **Tutting** — full-canvas LED grid + a line hopping between points in sharp 90° angles.
- **Strobe** — radial sun-rays + independently random-flashing beams.
- **Orbits** — spirograph light-painting (compound circular paths) filling the canvas.
- **Bloom** — an LED wall of color panels blooming in/out (Simon-style).
Beat energy is read from `hit` onsets (kick/snare/bass/lead/crash/bar) via `env()`. Recording of the
Glover canvas (MP4 on capable browsers, else WebM) is available; audio tapped from master.

---

## 7. NODE: TITLE GENERATOR

Generated loop names via `makeTitle()`. Word banks are **classed by provenance** (a hard rule — only
Wes-contributed terms go in his classes, Claude-added connective words are quarantined and labeled):
- **BRANDS** (Wes) — retired '90s/Y2K rave brands (G.A.T., MacGear, Caffeine, Stuka, …).
- **MONIKERS** (Wes) — DJ heroes, possessive (Coxy's, Fatboy's, Slim's, Sven's, Paul's).
- **VENUES** (Wes) — venues (Simon's).
- **FX_DESC / FX_END** (auto flavor, labeled) — connective descriptors/endings.
Combined into varied patterns ("Coxy's Warehouse", "Live at Simon's", "Phat Caffeine 47"). Fires on
every generate and party swap. The INFO genre field + `patName` update alongside.

---

## 8. LEAF REFERENCE (features, controls, keyboard)

### Transport (Controls → Transport module)
play/pause · randomize (⚄ generate) · kill (master mute — silences the mix without stopping
transport). (The old ⏭ next button was removed — redundant with generate, since there's no playlist.)

### Keyboard / WASD grid navigation
WASD navigates the control grid (rows: Controls, Voice selectors, Voice randoms, Voice kills, plus the
Master FX). A/D move within a row (wrapping across rows as one ring); W/S jump between rows (land at
the left). Arrow keys change the focused control's value. Tab is untouched (standard). Space = play,
G = generate, N = generate.

### Other leaves
Persistence (∞), orientation (16:9 / 9:16 for Glover recording), record, per-voice reverb, master FX
strength, quantize readout (Q ♩ display), party duration, time-slot selector.

---

## 9. THE AMU → PROFILE PIPELINE ("YO CONDITIONING")

**The concept has a name: YO DNA (the artifact) and YO CONDITIONING (the method).**
- **Yo DNA** = the extracted fingerprint of a style — the `profile:{}` object (mode, swing, structure,
  dynamics + v2 arrangement: densities, bass entry, vocal ratio, breakdowns). The *noun*.
- **Yo Conditioning** = the act of using that DNA to bias the generator toward the style. The *verb*.
  It's "corpus conditioning" — the actual ML term for biasing a generative system on extracted
  features — reworded as a ONEMANSHYO pun without losing technical accuracy. "Extract the Yo DNA from
  the catalog, then use it for Yo Conditioning on the generator" is a literally-correct sentence.

The lineage: **Dojo** (first, ~2025) established the OMS single-file/module/audio-engine framework and
vocabulary. **Sozo** is the analysis app — it wraps Apple's **Music Understanding** framework via the
**`omsanalyze`** Swift CLI (`MusicUnderstandingSession(asset:).analyze()`), emitting a JSON sidecar per
track (exact integer BPM, beats/bars as absolute timestamps, key as time-ranges, loudness LUFS/peak,
and song **structure** = sections/phrases/segments). Sozo also has an HTML visualizer for reading that
JSON (its visual grammar of squares/lines is why Chippy's matrix looks as it does).

**Chippy uses this for selecta profiles.** Workflow: run `omsanalyze` on reference tracks (e.g. Wes's
own catalog) → get JSON sidecars named to match each track → analyze *across* the set for the common
signature (mode, swing/groove, BPM range, structural bar-grid, loudness/dynamics) → encode that
signature as a selecta `profile:{}` (§3.1). This is not sampling or playback — it's learning the
*DNA* (fingerprint) of a style and teaching the generator to produce fresh patterns that carry it.
Juice Night Out (from 12 of Wes's 124–128 house tracks) is the proof-of-concept. **omsanalyze v2 now
emits real `instrumentActivity`** (bass/drum/other/vocal, each with per-time activity curves + ranges) —
this was previously stubbed/TODO but is live as of the v2 sidecars. Per-band DENSITY is therefore
AMU-derived and wired (§2.5). Per-drum onset PLACEMENT (exact kick step patterns) is still not extracted
— that's the micro layer MIDI DNA will supply. So profiles currently shape groove/key/structure/dynamics
AND per-band density, not yet exact kick patterns.

---

*Living document. Mirrors the OMS Dojo Developer Documentation framework (Cortex/Node/Leaf) at
Chippy's scale. Grows as Chippy matures.*
