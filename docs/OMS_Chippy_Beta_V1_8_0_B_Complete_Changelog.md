# OMS Chippy — Changelog

## V1_8_0 — Voices/Mixer split, module cleanup, Design System documented (2026-08-31)
Rolled up from iterations V1_8_01…V1_8_05 (the UI line following the V1.7.0 YoConditioner ship).
### Added
- **Mixer section** — split the per-voice **level + reverb + kill** out of Voices into a new **Mixer**
  row (the console/④ in the seven-section model). Voices now holds only the instrument controls
  (**selector + dice**); the Mixer holds the DJ's-hands controls. They line up lane-for-lane. This is
  the surface the YoConditioner's mixing moves will visibly act on (kickCut/bassEqOut/leadDrop/throw).
- **MASTER FX** moved from the Voices header into the **Mixer header** (where the console lives).
- **The Design System** — documented as a Cortex system in the Y-doc (System Reference): tokens,
  typography, layout/spacing, color semantics, component patterns, and the anti-drift hard rules.
  Previously only a 6-value palette existed; the full spec now exists so UI work has a written law to
  build against (this addition is recorded in the backlog as its own ticket).
### Changed
- **Module cleanup** — every module control (selector, level, dice, reverb, kill) is now a uniform
  30px-tall single row. Fixed truncated selectors ("909 Kic" → "909 Kick"), floating values, mismatched
  padding, and oversized boxes. Voices and Mixer modules are consistent, modular, clean.
- **Vertical squeeze** — trimmed section-header margins/height, strip padding, and the Info row
  (padding + face-canvas height) so the whole TRACKS tab fits one screen on a 14" MacBook Pro.
### Fixed
- iOS AirPlay routing (BLOG#0025) — verified working on device; Chippy audio now routes to
  HomePod/AirPlay via the `<audio>`-element output path. Ticket closed.
- Mixer header shipped magenta / MASTER FX rendered oversized / channel labels added — all corrected to
  match the (now documented) design system: cyan headers, small-gray master-FX label, no module labels.

## V1_7_0 — The YoConditioner: Yo DNA → Yo Conditioning wired to audio (2026-08-31)
Rolled up from iterations V1_6_01 and V1_6_02. First release where committed DNA audibly biases
generation beyond swing/mode — the YoConditioner is now a named Cortex-tier system.
### Added
- **The YoConditioner** — the conditioned generative layer, promoted to a canonical Cortex section
  (C-doc §2.5, Y-doc). Two primitives kept strictly distinct: **Yo DNA** (the noun — committed
  reference fingerprint) and **Yo Conditioning** (the verb — biasing the generator with it). Sits
  ABOVE the swappable sound set (currently chiptune); not training, not retrieval — it tilts a
  stochastic generative process, never writing the output.
- **Density conditioning (COMPOSITION DNA)** — the JNO profile's measured per-band densities
  (drumDensity 0.654, bassDensity 0.447, melodicDensity 0.452) now bias each band's fire probability,
  normalized against the busiest band (drums) and floored so no band vanishes. Drum-forward JNO ⇒
  drums full, bass/melodic thinned to their measured proportion. Code: `setDensityFromProfile`,
  `dGate`, `gk`.
- **Performance moves (MIXING DNA, authored)** — DJ gestures imposed on the generated track at 4-bar
  phrase boundaries, each gated by a probability so they're alive, not mechanical: **kickCut**,
  **bassEqOut**, **leadDrop**, **reverbThrow** — uniform baseline 25% chance, 1-note-to-1-bar length.
  Code: `buildMoves`, `moveAt`, `_moveWindows`, honored in `fireVoice`.
- **Tempo creep (MIXING DNA)** — `maxBpmJump` caps the tempo move between consecutive party loops
  (JNO = 2 BPM: build energy by nudging, not leaping). Code: the tempo block in `partyReroll`.
- **Corpus manifest** — the JNO profile now self-documents its Yo DNA source: the 12 AMU'd tracks
  (`corpus:[]`), so every profile records which references conditioned it.
- **Measured DNA** — profile values updated from real omsanalyze v2 `instrumentActivity` means across
  the 12-track corpus (previously hand-authored approximations): densities, bassEntry 0.066,
  vocalRatio 0.229, ~-9.8 LUFS, 12/12 minor, bpm 124–128.
### Changed
- **House-only focus** — genre dropdown collapsed to House; the techno/breaks/dnb/bigroom builders were
  removed from the code to cut clutter while the YoConditioner is dialed in on JNO. They return rebuilt
  to the new standard once House is proven (mirrors the V1.6.0 single-selecta move).
- **YoConditioner is party-gated** — the whole system applies ONLY under party mode. Plain generate
  gives raw stock house (neutral density, no moves); party applies the active selecta's DNA.
- Dead `PARTY_GENRES` array removed.
### Fixed
- Stale docs corrected: omsanalyze v2 **does** emit `instrumentActivity` (bass/drum/other/vocal with
  activity curves + ranges) — the "stubbed/TODO" claim in the C-doc §9 and Y-doc was outdated. Per-band
  density is now AMU-derived. (Per-drum onset PLACEMENT is still not extracted — that's the MIDI-DNA
  layer.)
### Captured but not yet wired (present in the profile, no conditioning consuming them yet)
- `bassEntry`, `breakdowns` (needs a threshold definition), `vocalRatio` (no vocal voice in the chiptune
  set), `structure`, `dynamics`, and mixing `approach`/`approachBars` (a true tempo glide of the outgoing
  loop needs its own scheduler-clock iteration).

## V1_6_0 — Juice Night Out Selecta / Single-Selecta Focus (2026-08-31)
### Changed
- **Selecta renamed** Giorgio Levan → **Juice Night Out** (id `jno`) — the selecta now models the
  real Juice Night Out label sound; the concept is literal (the selecta IS the sound of the label)
  rather than a namesake. Profile values (mode/swing/structure/dynamics + arrangement DNA) unchanged.
- **Single-selecta focus** — the other selectas (House, Battle, Open Format, Warehouse Rave, Jungle)
  were removed from the menu and `SELECTAS` array; only Juice Night Out ships, ahead of the refined
  Party Grid v2 work. Genre engine (house/techno/breaks/dnb/bigroom tempo ranges) is untouched.
- **Internal naming** `DJ_MODES` → `SELECTAS`, `djMode` → `selecta`, `djIdx` → `selectaIdx`, and the
  remaining "DJ" field copy → "selecta" (the "DJ heroes" title-generator terms are unrelated and kept).
- **Selecta field tooltip** rewritten to "pick the selecta that matches your vibe." with the
  navigation hint split onto its own line.

## V1_5_0 — Glover Light-Show, FX Voice & Party Grid Release (2026-08-30)
### Added
- **GLOVER tab** (renamed from RAVE ON) — full-screen light show, six tricks: Tracers (comets w/
  trails), Liquid (water wave-bands), Tutting (LED grid + 90° line), Strobe (rays + random flashing
  beams), Orbits (spirograph light-painting), Bloom (LED color wall). Chippy family always in front.
- **Voice 8 = FX** — effects voice with 7 types: Riser Up/Down, Glitch Up/Down (chopped stutter),
  Impact, Sweep, Ship Horn (detuned-saw foghorn). All generated, no files.
- **Per-voice reverb** button on each module; **Master FX** bus (reverb on whole mix except drums)
  with a strength value, in the VOICES header.
- **Jungle** selecta (pegged to D&B) and **Juice Night Out** selecta (funky/tech house).
- **Selecta profiles (DNA)** — AMU-derived `{mode,swing,structure,dynamics}` from analyzing real
  reference tracks. Juice Night Out built from 12 of Wes's own 124–128 house tracks.
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
