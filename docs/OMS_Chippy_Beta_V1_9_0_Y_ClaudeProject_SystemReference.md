# OMS Chippy [Beta V1_9_0] — SYSTEM REFERENCE

Machine-readable reference for an AI/dev picking up the single-file codebase. Names below are the
actual identifiers in the HTML's inline script.

## CONSTANTS / THEME
- `CY #00d4ff, MAG #ff006e, YEL #ffcc00, GRN #00e08a, PUR #c77dff, ORG #ffa500` — palette (voices/UI/Glover).
- `STEPS_PER_BAR = 16` (16th grid). `BARS` loop length; `TOTAL_STEPS = STEPS_PER_BAR*BARS`; `setBars(n)`.
- `LOOKAHEAD = 0.25` (scheduler lookahead seconds).
- `A4 = 440`; `midiToHz(m)`; `NOTE{}`, `MINOR[]` (natural minor); `keyTonic` (default "A", app is minor-locked).

## THE DESIGN SYSTEM (Cortex — universal UI/UX spec)

**Purpose:** the universal UI/UX rules for Chippy. Change any of these = audit the whole app. This is
CORTEX: it governs visual consistency across every section and tab. Deviation reads as amateur slop and
breaks the OMS look. **When building or moving any UI, inherit these tokens/classes — never hand-roll a
one-off.** (This section exists because unlisted tokens are how drift happens — a hand-rolled panel
picks the wrong color and nothing catches it.)

**Lineage:** Chippy inherits the OMS design language from the Sozo/Dojo template (dark rack aesthetic,
Eurorack-style module boxes, cyan section headers). It is a scaled-down sibling of the Dojo Design
System (which is also a Cortex system). Same language, Chippy's own values.

### Design Tokens (the real CSS values)
```
/* Section frame (Info/Controls/Voices/Mixer/Matrix panels + module boxes) */
background:  #0a0a0a        /* panel + module fill */
border:      1px solid #2a2a2a
radius:      6px (panels) · 5px (modules) · 4px (inner controls/buttons)

/* Inner control fill (selectors, level cells, buttons) */
control-bg:      #151515
control-border:  1px solid #333
control-height:  30px        /* EVERY module control is 30px tall — uniform rows */

/* Palette (JS consts CY/MAG/YEL/GRN/PUR/ORG) */
CY  #00d4ff  cyan     GRN #00e08a  green
MAG #ff006e  magenta  PUR #c77dff  purple
YEL #ffcc00  yellow   ORG #ffa500  orange
```

### Typography
- **Section headers** (`.sec-hdr .txt`): 11px, weight 600, uppercase, letter-spacing 1px, Courier New (mono).
- **Mono UI** (values, counters, selectors, buttons): `'Courier New', monospace`.
- **Display/body**: `'SF Pro Display', -apple-system, BlinkMacSystemFont, sans-serif`.
- **Module selector text**: 11px mono; must be fully readable (selector flex-grows to fit — never clip
  "909 Kick" to "909 Kic").

### Layout & Spacing
- **Platform target:** Chrome on MacBook Pro 14" — the whole TRACKS tab should fit one screen.
- **Section header** (`.sec-hdr`): margin `6px 0 4px`, min-height 18px, cyan bar (3×13px) + cyan
  uppercase label + a `#262626` rule line filling the row.
- **Module strip** (`.devstrip`): flex, gap 6px, padding 6px, margin-bottom 6px, wrap.
- **Module** (`.devmod`): padding 6px, min-width 120px, flex:1; a single 30px-tall control row.
- **Control heights are uniform 30px** across selector, level cell, and buttons so Voices and Mixer
  rows align lane-for-lane.
- **Button** (`.dm-btn`): 34px wide × 30px tall, `#151515`/`#333`, radius 4px.

### Color Semantics (each color MEANS something — not decorative)
| Color | Hex | Meaning / where |
|---|---|---|
| Cyan | `#00d4ff` | **Selection/focus + ALL section headers.** Every `.sec-hdr` is cyan. Info/Controls/Voices/Mixer/Matrix headers, focus rings, active tab. |
| Magenta | `#ff006e` | Kill/mute state + the party button + matrix playhead. |
| Green | `#00e08a` | Audio-effect ON (reverb sends, master FX `fxon` glow). |
| Yellow/Orange | `#ffcc00`/`#ffa500` | Accent/time-slot warmth; per-voice lane colors. |
| Purple | `#c77dff` | Per-voice lane color (synth-ish lanes). |
| Red | `#e0114b` | Engaged kill (solid-red `.killed` state). |

**Voice-lane colors** carry through matrix, modules, and Glover for one identity per voice.

### Component Patterns (the reusable furniture)
- **Section header** = `.sec-hdr` → `<span class="bar">` (colored 3px bar) + `<span class="txt">`
  (uppercase label) + `<span class="rule">` (filler line). Always cyan unless a section owns a color.
- **Module box** = `.devmod` inside a `.devstrip`. One `.dm-row` (30px) of controls.
- **Icon button** = `.dm-btn` (+ `.dm-rand` dice / `.dm-fx` reverb ring / `.dm-kill` power). States:
  `:hover` tints to the semantic color; `.fxon` = green glow; `.killed` = solid red.
- **Value wheel** = `.dm-wheel .wv` — scroll/drag numeric cell, 30px tall, centered value.
- **Master-FX cluster** (mixer header): small gray `.vh-mlabel` + green-ring `.vh-mfx` + value
  `.vh-mamt` + `.vh-mkill`. (Scoped to `#mixerHdr` — if this cluster moves, move its CSS scope with it.)

### Hard Rules (the anti-drift laws)
1. **Section headers are ALWAYS cyan** `#00d4ff`. Never recolor a header (this was violated once —
   a mixer header shipped magenta; corrected).
2. **Inherit existing classes; never hand-roll a parallel style.** New UI reuses `.devmod`/`.dm-btn`/
   `.sec-hdr`. Hand-rolled inline styles drift.
3. **No text labels on modules** except in the Matrix piano-roll lane chips. (Channel-name labels were
   added to the mixer once and removed — the lane alignment identifies the channel.)
4. **Uniform 30px control height** — selector, level, buttons all match so rows align.
5. **Buttons are square-ish** (34×30), `#151515`/`#333`. Not pills, not full-width.
6. **When a styled element moves between headers, move its CSS scope too** (the `#voicesHdr .vh-*` →
   `#mixerHdr .vh-*` bug: styles were ID-scoped and broke on the move).

## AUDIO NODES
- `actx` AudioContext; `master` master gain → destination.
- `noiseBuf` (white), `pinkBuf`, `brownBuf`.
- `reverbBus` (ConvolverNode, generated impulse) → `reverbWet` → `master`.
- `curDest` — per-fire routing target (master, or a tap that also sends to reverbBus).

## DATA CONTAINERS
- `VOICES[]` — 8 voices `{id,name,color,osc,on,lvl,decay,...}`: drums, perc, bass, lead, synth, accent, fills, fx.
  `V{}` = id→voice. Voice ids also index pattern arrays `P{}`.
- `GENRE{}` — per-genre `{name, bpm, tempo:[min,max], build:fn}`. **House-only as of V1.8.0** (techno/
  breaks/dnb/bigroom builders REMOVED while the YoConditioner is dialed in on JNO; return rebuilt later).
- `SELECTAS[]` — selectas: `{id,name,genreChance,barsAllowed[],barsChange,pool[],bpm:[min,max], corpus[]?, profile?}`.
  Ships two: `none` (index 0 — raw play, `profile:null`, no conditioning/auto-swap) + `jno`
  (Juice Night Out, index 1). `selecta()` = active; `selectaIdx` (0 = None). The party button was
  retired V1.9.0 — selecting a non-None selecta is the mode switch (`yoOn`); None stops the set.
- `VIBES[]` — time slots: `{id,name,barBias,density,level}`: opener, support, headliner, closer.
- Selecta `profile:{}` — the Yo DNA (see YOCONDITIONER below).
- `OSC_OPTS`, `KICK_OPTS`, `FX_OPTS` — module dropdown option lists.
- Title banks: `BRANDS`, `MONIKERS`, `VENUES` (Wes-contributed), `FX_DESC`, `FX_END` (auto flavor).

## THE YOCONDITIONER (Cortex — the conditioned generative layer)
The system that decides WHAT gets generated, ABOVE the (swappable) sound set. Two primitives, distinct:
- **Yo DNA** (noun) — committed reference fingerprint. Lives in `profile:{}` (+ `corpus:[]` provenance).
  Source = anything Wes commits: AMU (omsanalyze), MIDI, DJ-mixes, authored knowledge.
- **Yo Conditioning** (verb) — biasing the generator with that DNA. Not training, not retrieval; it
  tilts a stochastic process, never storing output.

Applies ONLY when a selecta is active (`yoOn`, i.e. selecta ≠ None). None ⇒ raw stock house
(neutral gate, no moves). (`partyOn`→`yoOn` rename, V1.9.0.)

Profile schema (JNO):
```
profile:{
  // COMPOSITION DNA (AMU-measured, omsanalyze v2 instrumentActivity means across the 12-track corpus)
  mode:"minor", swing:[0,0.05], structure:[2,4,8], dynamics:"punchy",
  drumDensity:0.654, bassDensity:0.447, melodicDensity:0.452,
  bassEntry:0.066, vocalRatio:0.229, breakdowns:5,
  // MIXING DNA (authored — the performance layer, imposed on the generated track)
  mixing:{
    maxBpmJump:2, approach:"outgoing", approachBars:4,
    moves:{ kickCut:{on,chance,minLen,maxLen}, bassEqOut:{...}, leadDrop:{...}, reverbThrow:{...} }
  }
}
```
**Wired to audio (V1.8.0):** `mode`, `swing`; per-band DENSITY (`setDensityFromProfile`→`dGate`→`gk`,
applied at the house builder's band probability points); `maxBpmJump` (clamp in `yoReroll`);
performance MOVES (`buildMoves`→`_moveWindows`→`moveAt`, honored in `fireVoice`).
**Captured, not yet wired:** `bassEntry`, `breakdowns`, `vocalRatio`, `structure`, `dynamics`,
`approach`/`approachBars`.

## EVENT / PATTERN SHAPES
- Pattern `P = {drums:[],perc:[],bass:[],lead:[],synth:[],accent:[],fills:[],fx:[]}`.
- Event `{step, dur, note|null, accent?, snare?, crash?}`. `note` = MIDI or null (percussive).
  `step` survives into the flattened event (used by `moveAt` for move windows).
- `rebuildEvents()` flattens P into a time-sorted `events[]` (seconds-into-loop `.t`, plus `step`).

## KEY FUNCTIONS
- Synthesis: `playKick`, `playNoise`, `playPulse/playTone` (osc voices), `playSweep`, `playGlitch`, `playHorn`.
- Routing: `fireVoice(ev,when)` — sets curDest (per-voice/master reverb), applies move windows, dispatches.
- YoConditioner: `setDensityFromProfile(pf)`, `dGate(band)`, `gk(band,p)`, `buildMoves(pf,R)`, `moveAt(band,step)`, `_moveWindows`.
- Generation: `buildPattern(seed)` → sets density + moves (party) → `GENRE[genre].build(P,R)`; `makeTitle()`.
- Genre builders: `house` (only). `mkRnd(seed)` seeded RNG.
- Clock: `scheduler()`, `nextBeatCtx()`, `quantize()/quantizeKey()`, `applyPendingSwap()`.
- Set (ex-"party", V1.9.0): `startSet()`, `yoReroll()` (incl. maxBpmJump clamp), `stopSet()`, `applyDj()`
  (selecta pick → bar-boundary swap via `skipArmed` when playing), `selecta()`, `vibe()`.
- Clock: `sessionClock()` → `{es, esF, cycles}` (single continuous session clock; shared by the MATRIX
  readout and the roll's continuous-wrap gate).
- Render: `draw()` (matrix), `drawFace()`, `drawRave()` (Glover), `buildStrip()` (voice modules).

## FLOW
generate() → buildPattern(seed) → rebuildEvents() → draw() ; scheduler() fires events on the audio
clock. Set: pick selecta → startSet (or, while playing, stage via yoReroll + skipArmed) → build/condition
loop → yoReroll stages next → scheduler applies at the BAR boundary via applyPendingSwap.

## PARTY GRID OWNERSHIP
- SELECTA owns bars (barsAllowed+barsChange) + crate (pool+genreChance).
- GENRE owns tempo (GENRE[].tempo, the hard clamp). Selecta bpm biases within it.
- TIME SLOT owns level/density/pace (VIBES).
- PROFILE (Yo DNA) conditions composition (mode/swing/density…) + mixing (maxBpmJump/moves…) in party.

## CONTROL-BEHAVIOR CATEGORIES
RESTART (genre → quantizeKey, next beat) · CONTINUE (tempo, swing, voices → live) ·
BAR-BOUNDARY SWAP (selecta while playing → yoReroll+skipArmed, next bar line; selecta while stopped → immediate) ·
WAIT-FOR-LOOP (length → pendingLen, loop boundary).

## AMU SEAM (Sozo / omsanalyze)
Yo DNA is derived from AMU JSON sidecars (omsanalyze → MusicUnderstandingSession). JSON gives bpm,
beats/bars timestamps, key(time-ranges), loudness(LUFS), structure(sections/phrases/segments), and —
**omsanalyze v2 — real `instrumentActivity`** (bass/drum/other/vocal, each with activity curves +
ranges). Per-band DENSITY is therefore AMU-derived and wired. (Previously this was stubbed/TODO; v2
provides it.) Per-drum onset PLACEMENT (exact kick step patterns) is still not extracted — that's the
MIDI-DNA layer.

## CONSTRAINTS
Single-file, zero-dependency, offline, Chrome-targeted. No external fonts/frameworks/samples. All
synthesis generated. App is minor-locked. bars are fixed dropdown values (never multiplied off-grid).
