# OMS Chippy [Beta V1_0_0] — DEVELOPER DOCUMENTATION

**License:** GPL-3.0 · Single-file HTML · Chrome-targeted · no dependencies · no build

---

## ARCHITECTURE OVERVIEW

One IIFE, no modules. Data flows one direction:

```
GENRE builder → pattern (per-lane events) → rebuildEvents() → events[] (time-sorted)
      → scheduler() (lookahead) → fireVoice() → Web Audio
pattern → draw() → canvas piano roll
```

The generator invents notes; everything downstream is agnostic to where notes came from. This is the intended swap-in seam for a future MIDI parser: replace `buildPattern` output, keep the same event shape, and the scheduler / synth / roll are untouched.

## DATA SHAPES

- **Note event (per lane):** `{ step, dur, note, accent?, snare?, crash? }`
  - `step` — 0..63 (16 steps/bar × 4 bars)
  - `dur` — length in steps
  - `note` — MIDI number, or `null` for noise lanes (drums/perc/fills)
  - flags — `accent` (downbeat), `snare` (clap/snare burst), `crash` (phrase-top wash)
- **Scheduler event:** the above flattened into `{ t (sec in loop), voice, note, dur (sec), step, accent, snare, crash }`, sorted by `t`.

## KEY MODULES

- **Musical config** — `STEPS_PER_BAR=16`, `BARS=4`, `TOTAL_STEPS=64`; `BPM`, `SWING`; `secPerBeat/Step`, `loopLen`.
- **Pitch** — `MINOR` scale, `rootMidi()` (tonic offset from C3=48), `deg(degree, octave)`, `midiToHz`.
- **VOICES[]** — 7 lane configs `{id, name, color, osc, on, lvl, decay, noiseLo?, noiseHi?}`. `osc ∈ {pulse12, pulse25, pulse50, triangle, noise}`.
- **GENRE{}** — `{name, bpm, build}` per style. `buildPattern(seed)` dispatches to the selected `build(P, R)`.
- **Generators** — `mkRnd(seed)` xorshift RNG (re-rollable), `euclid(hits, steps)` boolean rhythm, `eachBar(fn)` helper.
- **Synth** — `makePulseWave(duty)` (Fourier PeriodicWave, cached), `playPulse/playTri/playNoise`, `playKick` (pitched sine sweep + click). `fireVoice(ev, when)` routes by voice/osc/flags.
- **Transport** — `startTransport/stopTransport/toggleTransport`; `scheduler()` lookahead (0.12 s) with loop wrap; `frame()` rAF drives scheduler + playhead + draw.
- **Canvas** — `draw()` renders lane backgrounds, grid, notes (pitch-positioned, clamped to lane band), labels (drawn last), playhead. `X(step)` maps step → x.
- **UI** — `buildStrip()` voice strip, `buildLegend()`, genre `<select>`, tempo/swing sliders, tabs.

## VOICE ROLES (consistent across genres)

drums = kick · perc = hats/snare (noise) · bass = triangle · lead = melody · synth = chords · accent = extra stabs · fills = noise turnaround.

## EXTENDING

- **New genre:** add an entry to `GENRE{}` with a `build(P,R)` that pushes events per lane. Anchor beat 1 (accented kick + crash) and end with `lastBarFill`.
- **New voice osc:** extend `OSC_OPTS`, handle it in `fireVoice`.
- **MIDI import (planned):** hand-rolled SMF parser (`DataView`) → voice-allocation adapter emitting the standard event shape → feed `pattern`. No changes to scheduler/synth/roll.

## EXPORT

`exportWav()` renders the loop in an `OfflineAudioContext` by temporarily pointing the module-level `actx`/`master`/`noiseBuf`/`pulseWaves` at the offline context, scheduling every event through the unchanged `fireVoice`, rendering, then restoring the live refs in a `finally`. The rendered buffer is hand-encoded to 16-bit mono PCM WAV (`encodeWav`) and downloaded (`downloadBlob`). No third-party encoder — single-file and offline.

## CHIPPY FACE

The Info-card face is a second canvas (`#face`) driven each frame by `drawFace()`. Reactions come from `detectHits(prevPos, pos, L)` in the frame loop, which stamps `hit.{kick,snare,bass,lead,crash,bar}` (performance.now ms) as the playhead crosses the corresponding events; `env(hitT, ms)` returns a 1→0 decay used to animate each feature (mouth open, squish, eye X, wink, brow, glance, crash sparkles). Stationary — only features animate; resting state is a plain grin.

## PARTY MODE

`startParty(mins)` sets `partyOn`, `partyEndsAt`, forces 64-bar songs, and calls `newPartySong()`, which picks a random genre, re-seeds, rebuilds the pattern (with `arrange()` applied since BARS>=16), and resets playback to the top. The frame loop computes `wrapped = p < prevPos` (a song/loop boundary) and calls `partyTick(wrapped)`: if the timer's expired it calls `stopParty()` (restores manual length + re-generates), else on a wrap it calls `newPartySong()` for the next song. Because each song is its own boundary, genre/tempo changes always land on the downbeat — no crossfade needed. `arrange(P)` filters events through an 8-section arc (`ARC[]`) that drops voices per 8-bar section to give a 64-bar song intro/build/drop/breakdown structure.

## CONSTRAINTS

Single-file, no dependencies, no build step. Web Audio + Canvas only. Browser storage APIs not used. AudioContext resumes on first user gesture (play).
