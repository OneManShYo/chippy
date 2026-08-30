# OMS Chippy — Changelog

## V1_5_0 — Glover Light-Show, FX Voice & Party Grid Release (2026-08-30)
### Added
- **GLOVER tab** (renamed from RAVE ON) — full-screen light show, six tricks: Tracers (comets w/
  trails), Liquid (water wave-bands), Tutting (LED grid + 90° line), Strobe (rays + random flashing
  beams), Orbits (spirograph light-painting), Bloom (LED color wall). Chippy family always in front.
- **Voice 8 = FX** — effects voice with 7 types: Riser Up/Down, Glitch Up/Down (chopped stutter),
  Impact, Sweep, Ship Horn (detuned-saw foghorn). All generated, no files.
- **Per-voice reverb** button on each module; **Master FX** bus (reverb on whole mix except drums)
  with a strength value, in the VOICES header.
- **Jungle** selecta (pegged to D&B) and **Giorgio Levan** selecta (funky/tech house).
- **Selecta profiles (DNA)** — AMU-derived `{mode,swing,structure,dynamics}` from analyzing real
  reference tracks. Giorgio Levan built from 12 of Wes's own 124–128 house tracks.
- **Random title generator** — classed word banks (BRANDS/MONIKERS/VENUES + auto flavor); genre field
  in INFO.
### Changed
- **Party Grid** formalized (selecta owns bars/crate, genre owns tempo, time slot owns level/pace).
  Selecta renamed; Warehouse+Rave merged; runaway-BPM fixed; party opens per-selecta bars & pool.
- Controls & matrix labels modularized (Eurorack boxes); About tab rebuilt; section spacing normalized.
- Transport = play/randomize/kill (redundant next button removed). FX curve = accelerating snap.
### Fixed
- Party opened on house regardless of selecta / on illegal 128 bars — now respects the selecta.
- FX audibility (low-pass not bandpass), level renormalization, numerous UI padding/alignment fixes.

## V1_3_0 — (2026-08-30)
Party-grid constraint matrix (selecta bar-sets, genre tempo bind), control-section Eurorack modules,
per-lane label wrappers, the three-category control-behavior model documented, session counter,
tempo/swing live-apply, length-at-loop-boundary.

## V1_1_0 — Performance & Recording Release (2026-08-29)
WASD grid navigation, arrow-key control on dropdowns, Ableton-style session counter, persistent record
cluster, long-recording fix (timeslice + real H.264/AAC-or-WebM), reserved 00:00 readout.

## V1_0_14 — Beat-Quantize Clean Cut (2026-08-29)
Control changes land cleanly on the next beat with no skip — beat-quantize anchored to a fixed
playback-origin grid so the boundary can't recede past the scheduler lookahead.

## V1_0_1 — (2026-08-29)
The ∞ button correctly labelled a persistence control (keeps playing across tab switches).

_Earlier V1.0 iteration history preserved in prior release bundles._
