# OMS Chippy — Changelog

## V1_14_0 — SID chip family + mobile Pocket Rave + Glover rename (2026-09-03)
Rolled up from iterations V1_13_01…V1_13_12 (the SID / mobile / rename line following V1.13.0). Live at chippy.onemanshyo.com.
### Added
- **SID 6581 voice-model family (Commodore 64).** A second chip model, swapped via the SOURCE dropdown
  (Ricoh 2A03 ↔ SID 6581). In SID mode every voice routes through the SID's signature multimode FILTER
  (BiquadFilter lowpass + resonance) and the triangle-bass becomes a sawtooth — turning the clean NES beeper
  into a fat, filtered, resonant C64 synth. First instance of the swappable voice-model-family architecture.
  Authentic quirk kept: the fixed filter doesn't keyboard-track, so some songs land the lead delightfully
  out-of-tune — exactly how real SID chips behaved (BLOG0107; see CODEX0005).
- **Mobile Pocket Rave — a portrait phone experience.** In portrait, Chippy shows a purpose-built one-tap
  surface instead of the desktop app (which stays for landscape/desktop): an animated Chippy that IS the play
  button (runs the default lineup, rolling a random chip per song), a Tron-neon monitor (info + roll-over
  now/next + a static loop⇢selecta-countdown clock), MUSIC/GLOVER tabs, a scrolling piano-roll behind the
  boxes, and a Moogerfooger enclosure (walnut cheeks, brushed-metal rails, a recessed top port-bay with a
  power switch + I/O jacks). GLOVER opens a true full-screen light show (real Tracers engine, edge-to-edge,
  tap to exit). Supersedes the old party-box (BLOG0005), folds in the rotate concept (BLOG0018) (BLOG0102).
- **Screen Wake Lock** — keeps the screen awake while playing so the phone's auto-lock never sleeps it and
  cuts the audio (BLOG0105).
- **Uniform voice-label chips** on the vertical roll — same-size marquee tags (BLOG0101).
### Changed
- **Visualizer renamed rave* → glover* throughout** — the internal engine was named "rave" (predating the
  GLOVER brand); standardized every identifier on Glover (the performer-role name, parallel to selecta), since
  "rave" is too generic. UI, code, and docs now agree (BLOG0103). Naming law recorded: "rave" is not a system
  name (BLOG0104).
### Fixed
- Mobile monitor marquee reset/scroll issues — calmed to roll-over/static, no runaway scrolling (BLOG0106).


## V1_13_0 — Resident DJ + the Lineup scheduler + monitor upgrades (2026-09-03)
Rolled up from iterations V1_12_01…V1_12_19 (the selecta/lineup/monitor line following the V1.12.0
MIDI-DNA release). Live at chippy.onemanshyo.com.
### Added
- **Resident DJ selecta** — the pre-JNO "Giorgio Levan" selecta (V1.5.0), flagged back per BLOG0026 and
  restored VERBATIM as a permanent house-act selecta named "Resident DJ": a minimal scalar-only profile
  (`mode:minor, swing:[0,0.05], structure:[2,4,8], dynamics:punchy`, bpm `[124,128]`, bars `[32,64]`), run
  through the V1.12.0 YoConditioner unchanged. Proves the conditioner works from a thin, corpus-free authored
  profile — no AMU/MIDI/stems required. `structure`/`dynamics` carried dormant (same as JNO). Default selecta
  on load (boots stopped; press play for a Resident DJ set) (BLOG0088, BLOG0091).
- **Lineup — session set-list scheduler** — a styled right-side slide-out (notepad button in the SELECTA row)
  to program a set: ordered slots, each a selecta + a duration in minutes. Play Lineup runs the slots in
  order, converting minutes→bars (`min·BPM/4`) and auto-advancing the selecta via the existing `applyDj`
  staged bar-boundary swap. Session-only (no persistence). A pure scheduler ABOVE the conditioner — no music-
  chain or clock contact (BLOG0093).
- **What's Next monitor** — a "playing" row in the MONITOR: a single-line scrolling now/next banner
  (selecta · genre · bpm · bars; None reads "None · raw play"). During a lineup it shows the upcoming DJ the
  whole time the current slot runs; on a manual swap it shows the cued loop (BLOG0090).
- **Countdown to next selecta** — the loop counter row (relabeled "loop"→"music") appends a third segment
  during a lineup: `⇢ b . b . 16`, the bars·beats·16ths remaining until the next slot, in the app's
  bar.beat.16th standard (BLOG0099).
- **Lineup running indicator** — a pulsing pink (`#ff006e`) glowing ring on the Lineup button while a lineup
  plays (design-system active-state idiom, not a dot), a header dot when open, and a moving marker on the
  currently-playing slot (BLOG0097).
- **Monitor + face hover hints** — every MONITOR row (master/music/tempo/playing) and Chippy's face now carry
  message-system hints (face: "Chippy — eat, sleep, rave, repeat") (BLOG0090, BLOG0099).
### Changed
- **Selecta-governed fields disabled under a selecta** — genre/bars/tempo/swing are overridden by the reroll
  while a selecta runs, so they're disabled (no-op made visible); time slot removed from the standard row
  (owned by the lineup). WASD nav skips disabled fields (BLOG0092, BLOG0094).
- **Lineup button relocated** to the SELECTA row; leftover native `title=` tooltips stripped from message-
  system controls (the Q/quantize one included) (BLOG0094).
### Fixed
- **Selecta-switch message** now fires when the swap actually lands at the bar boundary (not only when armed
  at click), so the message bar reflects the settled selecta instead of reverting to the resting hint
  (BLOG0089).
- **Minutes field** — stripped the native number-spinner arrows; design-system styling + scroll-to-change
  (BLOG0097).


## V1_12_0 — Yo DNA transplant + the section engine + the vertical channel-grid roll (2026-09-02)
Rolled up from iterations V1_11_01…V1_11_11 (the MIDI-DNA / section-engine / roll-orientation line
following the V1.11.0 mixer release). Live at chippy.onemanshyo.com.
### Added
- **MIDI DNA transplant (YoConditioner, L2b)** — the JNO selecta's `profile.midi.*` now carries Wes's
  actual programmed patterns, folded from the Ain't No Trick `.als` and seeded into the generator with
  optional-with-fallback reads: **kick** four-on-floor `[0,1,2,3]` (BLOG0045), **open hat** dead-straight
  offbeat, ghost suppressed (BLOG0046), **sub bass** E1 root rumble on `[0,3,6,8,11,14]` (BLOG0047),
  **clap** backbeat (BLOG0048). Source-nested under `profile.midi` to keep note DNA separate from the flat
  AMU-measured fields and from `profile.mixing` (BLOG0044).
- **The section engine (arrangement DNA → song shape)** — `profile.midi.arrangement` carries a per-song
  section map; `planSections(arrangement, BARS)` is the constant formula: a fixed *continuous-play factor*
  trims intro/outro (they only exist as mix headroom; Chippy never mixes), then the floor sections squeeze
  proportionally into the `BARS` budget. `yoArrange` applies the plan as probabilistic bias over the
  still-running generator — never a template, one of several DNA inputs (BLOG0051–0053, 0058).
- **Vertical channel-grid piano roll** — MATRIX roll toggles between horizontal (original) and a vertical
  orientation (time bottom→top, the 8 voices as columns aligned under the VOICES/MIXER grid). Toggle in the
  MATRIX header, GLOVER-style; default vertical. Overview strip + Chippy stay horizontal (BLOG0061).
- **Loop counter shows total** — the loop readout now reads current-over-total (e.g. `9 . 1 . 1 / 64 . 4 . 4`)
  so loop length is visible at a glance without hunting the bars field (BLOG0056).
### Fixed
- **yoArrange RNG scope crash** — selecting JNO blanked the whole matrix: `yoArrange` used the seeded RNG
  `R` out of scope; passing `R` in fixes it. None mode was unaffected, so it only surfaced on the JNO switch
  at ≥16 bars (BLOG0055).
- **None→JNO master-clock desync** — a length-changing selecta swap re-based `loopStartCtx` but not
  `playOriginCtx`, so the continuous clock (playhead + scroll) drifted from the audio by the swap offset. Fix:
  a swap is a re-origin — reset both anchors equal at the swap (as at play), restoring the documented
  single-clock two-view model (BLOG0057–0059). See CODEX0002.

## V1_11_0 — The performance mixer: audio-effects rack + master processing (2026-09-01)
Rolled up from iterations V1_10_03…V1_10_26 (the mixer/performance line following the V1.10.0 UI ship).
Live at chippy.onemanshyo.com.
### Added
- **Master audio-effects rack** (MIXER header) — four live performance effects on the whole mix, each a
  labeled button + 0–10 wheel: **Reverb** (send; drums stay dry; on by default, light), **Echo**
  (tempo-synced 3/16 delay with a low-cut feedback loop; wet returns in parallel — no runaway), **Filter**
  (bipolar DJ sweep — down = low-pass/underwater, up = high-pass/bass-out, 5 = neutral; button is a bypass
  toggle that keeps the sweep setting), **Backspin** (tape-stop brake — a rising-delay Doppler pitch-drop +
  lowpass + gain sag; momentary hold via pointer OR Enter/Space; the transport clock is untouched so
  release resumes on the live grid) (BLOG0044, 0056–0058).
- **Master channel processing** (MIXER header, end of the master bus) — set-and-forget session tone, each
  processor a self-contained `.mproc` module (future slide-out anchor): **EQ3** (High/Mid/Low, ±12 dB,
  lowshelf 250 / peaking 1k / highshelf 2500), **Compressor** (button + ratio macro 2:1→8:1),
  **Limiter** (button, −3 dBFS/20:1 peak ceiling), **Master Volume** (final live output level). Ported
  from the Dojo V5_3_38d DSP recipe. Everything defaults transparent (BLOG0061–0064).
- **Master Solo** (MIXER header) — a performance *release*: lights when any channel is soloed, one tap
  clears all solos (subtractive-build → slam back on the drop). Sits left of master volume (BLOG0044, 0065).
- **Keyboard message-bar parity** — the footer bar now updates on keyboard/controller **focus**, not just
  mouse hover; re-wired to survive strip rebuilds (BLOG0053).
- **jsdom boot-check discipline** — every UI-halting change is headless-boot-verified before ship
  (`node --check` catches syntax only, not runtime TDZ) (BLOG0053/0059).
### Changed
- **Audio-effects family renamed** — every `fx`/`mfx`/`Fx` token in the master-effects code replaced with
  spelled-out `audioEffects*`/`master*`. **FX** (capital) is reserved for the FX *voice* (a sound in the
  matrix); audio effects are *processing*. No `fx` anywhere near the processing family (BLOG0055).
- **WASD grid centralized** — one authored `navRows()` descriptor is the single source of grid truth,
  reaching every editable field in visual (DOM) order; adding a control needs one edit in one place
  (BLOG0047/0048). Value wheels take horizontal touch-drag (mouse keeps vertical) (BLOG0046).
- **Mixer row layout** — per-control labels (REVERB·ECHO·FILTER·BACKSPIN·EQ·COMP·LIMIT·VOL) replacing the
  single AUDIO EFFECTS label; all wheels normalized to a 0–10 scale (EQ in dB); compact value boxes; EQ
  ordered High·Mid·Low (BLOG0064).
- **INFO block dedupe** — `source` row → `seed` (bare hex); `length` drops the fixed `steps/bar` (BLOG0045).
- **FX voice default → Glitch Down** (was Riser Up); vestigial **"Sweep FX"** removed from the tonal-voice
  source dropdown (it duplicated the real FX-voice Sweep and made no sense on melodic voices) (BLOG0051/0052).
- **About panel** — PLUR centered on the page with a divider; new **ASSETS** section carving the
  Squint/Wowzer icons, music, and MIDI (Juice Recordings LLC) out from the GPL-3.0 (source-code-only)
  (BLOG0049).
- **YoConditioning — audio effects nulled** — the `reverbThrow` move (the one auto-applied audio effect)
  is parked; composition moves (kickCut/bassEqOut/leadDrop) stay. Auto-driving the now-full effects rack
  waits until the effects are better understood (BLOG0066).
### Fixed
- **Boot-crash (TDZ)** — the keyboard-parity work first crashed boot (a `const MSG` accessed before init →
  empty mixer/blank matrix); fixed with a boot-ready gate. Same class as the V1.9.x HINTS crash (BLOG0053/0039).
- **Touch wheels** — value wheels were unusable on touch (vertical drag hijacked by page scroll); fixed
  with horizontal drag + `touch-action` (BLOG0046).
- **EQ dead controls** — the EQ wheels didn't bind (a missing `data-` prefix in the selector) (BLOG0063).
- **Filter default** — no longer armed/lit at boot (BLOG0066).


## V1_10_0 — Message System, Monitor, and UI structure pass (2026-09-01)
Rolled up from iterations V1_9_01…V1_9_24 (the UI/feedback line following the V1.9.0 selecta ship).
### Added
- **Message System (CORTEX)** — Dojo's message bar ported: a footer `#messageBar` and one entry point
  `showMessage(text, type, onClick)`. Hover-driven (phase 1 = `info` help on hover; phase 2 = `state`
  value on change; `error` = failures). Dojo colors verbatim: info cyan `#00d4ff`, state orange
  `#ffa500`, error magenta `#ff006e`. All feedback flows through it; no native `title=` tooltips remain
  (BLOG0038/0039/0040).
- **Central `MSG` registry** — single source of truth for every user-facing string: `MSG.hint` (hover),
  `MSG.state` (value-change, as functions of the new value), `MSG.error`. One place to change any
  message; the docs' field/label table is generated from it (BLOG0040).
- **MONITOR** — the center box of the top row rebuilt as a live realtime readout: master clock, loop
  clock, tempo, in the INFO box's grid grammar. Distinct from INFO (static description). Loop clock
  blinks through its last bar (BLOG0035/0030).
- **SOURCE readout** (VOICES header) — names the hardware the synthesis is modeled after: Ricoh 2A03
  (NES/Famicom). "Source" not "chip" so it stays correct for a future non-chip model (BLOG0034).
- **Per-channel SOLO** (MIXER) — replaces the per-channel reverb send; standard behavior (any solo
  silences non-soloed channels), gated at `fireVoice` (BLOG0032).
- **Canonical area names + internal handles** — INFO/MONITOR/FACE/TRANSPORT/SELECTA/VOICES/MIXER/MATRIX,
  each with a code handle, for docs + reference (BLOG0036).
### Changed
- **TRACKS tab renamed → MUSIC** (label only; internal wiring unchanged) (BLOG0037).
- **CONTROLS row split into two peer sections** — TRANSPORT (play + generate) and SELECTA (name, genre,
  time slot, bars, tempo, swing, quantize), two labelled sections sharing one header row (BLOG0033).
- **MIXER header relabeled** — "MASTER FX" → **AUDIO EFFECTS**; master kill labeled **MUTE**; transport
  kill removed (master mute now lives only on the mixer) (BLOG0031).
- **FX vs Audio Effects** distinction locked — FX = the FX voice (piano-roll lane 8); Audio Effects =
  the master reverb/processing bus. Labels/hints made consistent (BLOG0041).
- **Design-system color cleanup** — field value text set to neutral `#bbb` (selecta fields, SOURCE,
  audio-effects strength); cyan reserved for headers, focus, and the live MONITOR clocks (BLOG0041).
### Fixed
- **Overview Chippy rider** no longer vanishes early or jumps at the loop tail — it now reads the master
  loop clock (`masterLoopClock`) like the rest of the visuals, wrapped so it finishes the loop instead
  of resetting ~250ms early on the scheduler's lookahead (BLOG0029).
- **Matrix-header clocks** split into two independent readouts (loop + session); removed an un-specked
  "yo m:ss" counter; loop clock blinks on its last bar (BLOG0028).


## V1_9_0 — Selecta-driven playback + Master-Clock/visual fixes (2026-08-31)
Rolled up from iterations V1_9_01…V1_9_012 (the transport/clock line following the V1.8.0 Mixer ship).
### Added
- **None selecta** — a first-class `none` entry (index 0) meaning raw looping play: no conditioning,
  no auto-swap, honors the manual controls. Juice Night Out is now index 1.
- **`sessionClock()`** — one continuous session clock (`{es, esF, cycles}` from
  `actx.currentTime − playOriginCtx`) shared by the MATRIX readout AND the roll's continuous-wrap gate.
  Single source of truth; not a second clock (BLOG0006/0011).
- **Bar-boundary selecta swap** — a fourth control-timing category: changing selecta while playing lets
  the current bar finish, then drops the new selecta in on the bar line (the DJ "let the record run out"
  model), reusing the previously-unwired `skipArmed` scaffolding (BLOG0003).
### Changed
- **Party button retired → selecta IS the mode switch** — removed the separate party button; selecting a
  selecta engages the conditioned set (`yoOn`), None stops it. One less control in the controls row.
  Internal `party*` symbols renamed `yo*` (`yoOn`/`yoStart`/`yoTick`/`yoReroll`/`pendingYo`); on-screen
  "party m:ss" → "yo m:ss" (BLOG0002).
- **Piano roll is continuous once playing** — the roll now wraps so the loop's own head scrolls in at the
  tail; never blanks after first play. Real start point preserved: forward wrap always, backward wrap only
  after one completed cycle (BLOG0013).
- **Both MATRIX clocks read 1 . 1 . 1 at start** and share `bar . beat . sixteenth` format; dropped the
  hardcoded "bar N/64".
- **Fixed play/pause button width** — `#playBtn` is a fixed 42px so the ▶ → ❚❚ glyph swap no longer
  resizes the button and shifts the controls row.
### Fixed
- **Master-Clock/visual coherence cluster (BLOG0006)** — all rooted in the visual playhead/roll deriving
  from the scheduler clock anchors:
  - Pause/stop now truly resets to bar 1 · beat 1 (was freezing the readout mid-loop as a phantom
    resume-position); `stopTransport` zeroes `posSec`/`prevPos`/`playOriginCtx`/`loopStartCtx` + pending queue.
  - Playhead no longer pre-rolls through the loop tail before audio (pre-origin position clamped to 0
    during the deliberate ~60ms start lead).
  - First-ever play no longer has ~a beat of silence — the MediaStream `<audio>` bridge is warmed at
    audio-create time + a 180ms cold-start origin lead (the gap was pre-existing, exposed once the
    pre-roll was fixed; the audio start path itself was unchanged from V1.8.0).
  - No more visual JUMP at each loop boundary — the scroll frame + playhead run off the continuous session
    step (`esF`), not the per-loop-wrapped `posSec`.

## V1_9_0 — Voices/Mixer split, module cleanup, Design System documented (2026-08-31)
Rolled up from iterations V1_9_01…V1_9_05 (the UI line following the V1.7.0 YoConditioner ship).
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
