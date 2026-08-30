# OMS Chippy [Beta V1_2_0] — CLAUDE PROJECT INSTRUCTIONS

**License:** GPL-3.0 · Development protocols & principles

---

## WHAT CHIPPY IS

A browser-native chiptune EDM generator. Independent sibling app to OMS Dojo and Sozo. Built solo by Wes Smith under GPL-3.0. Single-file HTML, Chrome-targeted, no dependencies, no build. It is a **source** (generates audio), not audio-reactive — the opposite data direction from Dojo/Sozo.

## CORE PRINCIPLES

- **Single-file, no dependencies, no build.** Never split, never add a framework, package, or bundled library. (An MP3 export using inlined lamejs was tried in V0_3_1 and reverted in V1_2_0 to keep this principle; export is native WAV.)
- **Dance-only.** Chippy generates EDM. Every genre anchors beat 1 (accented kick + phrase-top crash, clap/snare on 2 & 4, last-bar fill). No "random notes in key" — patterns must read as their genre.
- **In-key by construction.** All pitches come from the scale via `deg()`. No accidental notes possible.
- **One event shape.** Generators emit `{step,dur,note,accent?,snare?,crash?}`. Scheduler, synth, and roll stay agnostic to the source — this is the MIDI swap-in seam.
- **GPL-3.0, free and open.** No paywalls, no monetization.

## VERSIONING

- Format `V#_#_#` (Major_Minor_Patch), underscores.
- Iterations at the current minor (e.g. V0_1_0 → V1_2_0); bug fixes edit in place, features bump the patch letter/number.
- Deliverables packages wrap an iteration series into the next version. (V1_2_0 is the debut package — build and package number aligned; the wrap-into-next cadence starts after.)
- HTML carries current version only (title, header, about, status). No changelog in the HTML.

## BUILD PROTOCOL

- Discuss/analyze freely; **ask before building.** Wait for "go" / "yes".
- Surgical changes; don't touch unrelated code.
- Always deliver the HTML + an MD notes file, and update the in-app version header.

## ADDING A GENRE

Add to `GENRE{}` a `{name, bpm, build(P,R)}`. The builder pushes per-lane events, anchors beat 1, and ends with `lastBarFill`. Keep voice roles consistent: drums=kick, perc=hats/snare, bass=triangle, lead=melody, synth=chords, accent=stabs, fills=noise.

## ROADMAP

- SMF (MIDI) parser → voice-allocation adapter → feed `pattern` (same event shape).
- Then a user-MIDI input tab.
- Deeper generation (Markov melody, structural re-rolls), per-genre swing/preset tuning, voice-strip discoverability.

## TERMINOLOGY

- Companion apps: **Dojo** (performance), **Sozo** (AMU visualizer), **Chippy** (this).
- **Style / Genre** — the four dance templates.
- **Lane / Voice** — one of the seven instrument rows.
- **Roll** — a generated pattern (a loop).
