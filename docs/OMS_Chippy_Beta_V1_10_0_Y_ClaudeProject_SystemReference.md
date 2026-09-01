# OMS CHIPPY [Beta V1_10_0] — SYSTEM REFERENCE

**Run date:** 2026-08-31  
**Current Unix Epoch:** 1788134400  
**App Version:** Chippy Beta V1_10_0  
**License:** GPL-3.0  

**Purpose:** Machine-readable index for an AI/dev picking up the single-file codebase. Names below are the
actual identifiers in the HTML's inline script.

**This is the "where is it" doc.** Its companion, the **C Developer Documentation**, is the "how it works"
doc. They do NOT overlap: C explains a system in prose once; Y lists that system's identifiers once and
points back to the C section that explains it. When a value or rule is documented here, C does not repeat
it, and vice-versa.

> **DOCUMENT TYPE — north star (read before editing).**
> **System Reference** is a *precise, dictionary-like listing* of the technical specifications —
> identifiers, schemas, parameters, values — that is **highly structured, concise, and meant for lookup
> rather than reading**. That is this document (Y).
> Its companion (C) is **Developer Documentation**: a holistic, narrative guide that *teaches* how the
> software works — read it to understand a system.
> *Adaptation for Chippy:* a classic system reference documents a public API; Chippy is a single-file app
> with no public interface, so Y indexes the **internal code identifiers** (the inline-script symbols).
> Reference in form (structured, lookup-oriented); internal codebase in subject.
> **The editing rule this implies:** entries here are *identifier + one-line "what it is" + pointer to the
> C section that explains it* — no teaching paragraphs. The moment an entry starts explaining *why* or
> *how* a system works, that prose belongs in C, not here. A one-line "what it is" is fine and expected;
> a paragraph of reasoning is the tell you're writing C content in the wrong doc.

---

# CORTEX SYSTEMS INDEX

The load-bearing systems, tagged for targeted extraction. The label is used sparingly: exactly one system
is Cortex (Timing). The other rows are the critical stack that sits on top of it (see C §2) — listed here
for the dependency map, not because they carry the Cortex label.

| # | System | Tag | Tier | Governance scope |
|---|--------|-----|------|------------------|
| 1 | **Timing (Master Clock + Scheduler)** | `MASTER_CLOCK` | **CORTEX** | `actx.currentTime`, the two clock accessors, anchors, scheduler, quantize, session lifecycle |
| 2 | Sound generation | `AUDIO_ENGINE` | critical | audio nodes, synths, routing, reverb bus |
| 3 | Composition | `COMPOSITION` | critical | pattern build, genre builders, accents, arrange, seeded RNG, event flatten |
| 4 | Conditioning (YoConditioner / Yo DNA) | `YOCONDITIONER` | rides on 3 | `profile:{}`, density + moves wiring, corpus |
| 5 | **Message System** | `MESSAGE_SYSTEM` | **CORTEX** | `showMessage()`, `#messageBar`, hover `data-hint` contract, types, sole feedback channel |
| — | Design System | `DESIGN_SYSTEM` | governed (not Cortex) | tokens, type, color semantics, component classes, anti-drift rules |

Dependency order (bottom-up): Timing → Sound generation → Composition → Conditioning. Only Timing is
Cortex; it sits on nothing. See C §2 for the explanation of each layer.

---

# CONSTANTS / THEME
<!-- @tags: MASTER_CLOCK -->
- `STEPS_PER_BAR = 16` (16th grid). `BARS` = loop length; `TOTAL_STEPS = STEPS_PER_BAR*BARS`; `setBars(n)`.
- `LOOKAHEAD = 0.25` (scheduler lookahead seconds).
<!-- @/tags -->
- `A4 = 440`; `midiToHz(m)`; `NOTE{}`, `MINOR[]` (natural minor); `keyTonic` (default "A", app minor-locked).
- Palette consts: `CY #00d4ff, MAG #ff006e, YEL #ffcc00, GRN #00e08a, PUR #c77dff, ORG #ffa500`
  (voices/UI/Glover). Full color semantics + tokens under THE DESIGN SYSTEM below.

---

# 1. TIMING — MASTER CLOCK + SCHEDULER  (CORTEX)
<!-- @tags: MASTER_CLOCK -->
Explained in C §2.1. One time source (`actx.currentTime`); two anchors give two views; every loop-phase
visual reads one function. Identifiers:

**Anchors / state**
- `playOriginCtx` — fixed session anchor, set once at play, reset to 0 on stop. Continuous view + quantize grid.
- `loopStartCtx` — current-loop start; advanced by the scheduler at every real loop boundary (natural wrap,
  bar-boundary selecta swap, quantized restart). Loop-phase view.
- `posSec` — scheduler-side loop position (for hit detection); clamps `<0 → 0` by design (grid-accurate).
- `prevPos` — previous `posSec` (hit-window edge). `audioColdStart` — first-start warm-up flag.

**Accessors (the two views)**
- `sessionClock()` → `{ es, esF, cycles }` — CONTINUOUS. `es` whole 16th-steps since play (never wraps),
  `esF` continuous fraction, `cycles = floor(es/TOTAL_STEPS)`. Read by: session readout, roll scroll +
  continuous-wrap gate, playhead.
- `masterLoopClock()` → `{ loopSec, loopStep, loopFrac, bar, beat, six, L, TOTAL_STEPS }` — LOOP-PHASE
  master. `loopSec = ((currentTime - loopStartCtx) % L + L) % L` (modulo-wrap, NOT clamp), held 0 when
  `currentTime < playOriginCtx`. Read by: Chippy rider, loop readout + last-bar blink, face bob (`drawFace`
  + `drawChippyAt`). THE single loop-phase source of truth — golden rule: all loop-phase consumers read it.

**Scheduler**
- `scheduler()` — lookahead loop; `horizon = now + LOOKAHEAD`; fires events via `fireVoice(ev, loopStartCtx+ev.t)`;
  owns every `loopStartCtx` advance; applies pending quantize / selecta swaps / length changes at their boundary.
- `nextBeatCtx()` — next beat after `now`, from `playOriginCtx`.
- `quantize(fn)` / `quantizeKey(key, fn)` — defer a change to the next beat. `applyPendingSwap()` — swaps
  the staged loop in at a boundary. `pendingQuant`, `quantTargetCtx`, `pendingLen`, `pendingYo`, `skipArmed`.
- `bgPump` — silent keep-alive node (backgrounded tab); toggled by ∞.
- Lifecycle: `startTransport()` (sets anchors; 60 ms lead, 180 ms cold), `stopTransport()` (zeroes anchors,
  clears pending — full reset to 1.1.1).

**Control-behavior categories** (C §2.1.7): RESTART (genre → `quantizeKey`) · CONTINUE (tempo, swing,
voices → live) · WAIT-FOR-LOOP (length → `pendingLen`) · BAR-BOUNDARY SWAP (selecta while playing →
`yoReroll` + `skipArmed`, next bar line; while stopped → immediate).
<!-- @/tags -->

---

# 2. SOUND GENERATION  (critical)
<!-- @tags: AUDIO_ENGINE -->
Explained in C §2.2. Identifiers:
- `actx` AudioContext; `master` master gain → destination.
- Buffers: `noiseBuf` (white), `pinkBuf`, `brownBuf`.
- `reverbBus` (ConvolverNode, generated impulse) → `reverbWet` → `master`. `curDest` — per-fire routing
  target (master, or a tap that also sends to reverbBus).
- Synthesis fns: `playKick`, `playNoise`, `playPulse` / `playTone` (osc voices), `playSweep`, `playGlitch`,
  `playHorn`.
- `fireVoice(ev, when)` — sets `curDest` (per-voice / master reverb), applies conditioning move windows,
  dispatches to the right synth. `rootMidi()` — track root for pitched FX.
<!-- @/tags -->

---

# 3. COMPOSITION  (critical)
<!-- @tags: COMPOSITION -->
Explained in C §2.3. Runs with conditioning OFF or ON. Identifiers:
- `buildPattern(seed)` → `mkRnd(seed)` seeded RNG → (if `yoOn`: `setDensityFromProfile` + `buildMoves`) →
  `GENRE[genre].build(P,R)` → `phraseAccents(P,R)` → (`arrange(P)` when `BARS≥16` and no selecta).
- Genre builders: `house` only (techno/breaks/dnb/bigroom retained, unlisted — see C §2.3).
- `rebuildEvents()` — flattens `P` into time-sorted `events[]` (`.t` seconds-into-loop, `.step`).
- `makeTitle()` — loop name (see §8 title banks).

**Pattern / event shapes**
- `P = { drums:[], perc:[], bass:[], lead:[], synth:[], accent:[], fills:[], fx:[] }`.
- Event `{ step, dur, note|null, accent?, snare?, crash? }`. `note` = MIDI or null (percussive). `step`
  survives into the flattened event (used by `moveAt` for move windows).
<!-- @/tags -->

---

# 4. CONDITIONING — YOCONDITIONER / YO DNA  (rides on §3)
<!-- @tags: YOCONDITIONER -->
Explained in C §2.4. Applies ONLY under `yoOn` (selecta ≠ None). Identifiers:
- Functions: `setDensityFromProfile(pf)`, `dGate(band)`, `gk(band,p)`, `buildMoves(pf,R)`, `moveAt(band,step)`,
  `_moveWindows`.
- Wired to audio (V1.8.0): `mode`, `swing`; per-band density (`setDensityFromProfile`→`dGate`→`gk`);
  `maxBpmJump` (clamp in `yoReroll`); performance moves (`buildMoves`→`_moveWindows`→`moveAt`, honored in
  `fireVoice`).
- Captured, not yet wired: `bassEntry`, `breakdowns`, `vocalRatio`, `structure`, `dynamics`,
  `approach`/`approachBars`.

**Profile schema (JNO)**
```
profile:{
  // COMPOSITION DNA (AMU-measured, omsanalyze v2 instrumentActivity means across the 12-track corpus)
  mode:"minor", swing:[0,0.05], structure:[2,4,8], dynamics:"punchy",
  drumDensity:0.654, bassDensity:0.447, melodicDensity:0.452,
  bassEntry:0.066, vocalRatio:0.229, breakdowns:5,
  // MIXING DNA (authored — performance layer imposed on the generated track)
  mixing:{
    maxBpmJump:2, approach:"outgoing", approachBars:4,
    moves:{ kickCut:{on,chance,minLen,maxLen}, bassEqOut:{...}, leadDrop:{...}, reverbThrow:{...} }
  }
}
```
- `corpus:[]` — provenance list of reference tracks that conditioned the profile.
<!-- @/tags -->

---

# 4B. MESSAGE SYSTEM  (CORTEX)
<!-- @tags: MESSAGE_SYSTEM -->
Explained in C §2.5. Hover-driven feedback bar (Dojo two-phase). NOT an action log.
- `showMessage(text, type, onClick)` — THE entry point → footer `#messageBar`. `onClick` optional (clickable).
- Two-phase: phase 1 ENTRY/hover = `info` (what a control IS); phase 2 CHANGE = `state` (the new value).
- Types (Dojo colors, verbatim): `info` cyan #00d4ff · `state` orange #ffa500 · `error` magenta #ff006e.
- CSS: `.message-bar` + `.msg-info`/`.msg-state`/`.msg-error` (+ `.clickable`); centered, bottom of `.app`.
- `setMsg(text,color)` — legacy shim (cyan/none→info, else→state).
- Central registry `MSG` = single source of truth: `MSG.hint{}` (phase-1 hover, keyed by control),
  `MSG.state{}` (phase-2 value-change, functions of the new value), `MSG.error{}`. Helpers:
  `msgHint(key)`, `msgState(key,...vals)`, `msgError(key)`. No message strings anywhere else; no
  native `title=` tooltips. Hover wiring at boot-end: `data-hint="<key>"` (labels) + id/`data-*` keys.
  Full field/label table in C §12, generated from MSG.
<!-- @/tags -->

---

# 5. DATA CONTAINERS
- `VOICES[]` — 8 voices `{id,name,color,osc,on,lvl,decay,...}`: drums, perc, bass, lead, synth, accent,
  fills, fx. `V{}` = id→voice. Voice ids also index pattern arrays in `P`.
- `GENRE{}` — per-genre `{name, bpm, tempo:[min,max], build:fn}`. House-only as of V1.8.0.
- `SELECTAS[]` — `{id,name,genreChance,barsAllowed[],barsChange,pool[],bpm:[min,max], corpus[]?, profile?}`.
  Ships two: `none` (index 0 — raw play, `profile:null`, no conditioning/auto-swap) + `jno` (Juice Night
  Out, index 1). `selecta()` = active; `selectaIdx` (0 = None). Selecting a non-None selecta is the mode
  switch (`yoOn`); None stops the set. (Party button retired V1.9.0.)
- `VIBES[]` — time slots `{id,name,barBias,density,level}`: opener, support, headliner, closer.
- `OSC_OPTS`, `KICK_OPTS`, `FX_OPTS` — module dropdown option lists.
- Title banks: `BRANDS`, `MONIKERS`, `VENUES` (Wes-contributed), `FX_DESC`, `FX_END` (auto flavor).

---

# 6. PARTY GRID OWNERSHIP
Explained in C §3. Ownership map:
- SELECTA owns bars (`barsAllowed` + `barsChange`) + crate (`pool` + `genreChance`).
- GENRE owns tempo (`GENRE[].tempo`, the hard clamp); selecta `bpm` biases within it.
- TIME SLOT owns level/density/pace (`VIBES`).
- PROFILE (Yo DNA) conditions composition (mode/swing/density…) + mixing (maxBpmJump/moves…), under `yoOn`.

Set functions: `startSet()`, `yoReroll()` (incl. `maxBpmJump` clamp), `stopSet()`, `applyDj()` (selecta
pick → bar-boundary swap via `skipArmed` when playing; immediate when stopped), `selecta()`, `vibe()`.

---

# 7. RENDER
- `draw()` — matrix + overview strip (Chippy rider reads `masterLoopClock`; playhead + scroll read
  `sessionClock`). `drawFace()` — INFO face canvas. `drawChippyAt()` — a Chippy head (Glover kids reuse it).
  `drawRave()` — Glover visualizer. `buildStrip()` — voice modules. `updateTime()` — the two MATRIX
  readouts (loop via `masterLoopClock`, session via `sessionClock`).
- Matrix geometry: `#tl` canvas; `LANEH`, `ROLLH = RPAD*2 + LANEH*8`, `RPAD` inset.
- Glover: `gloverTrick` (Tracers/Liquid/Tutting/Strobe/Orbits/Bloom); `hit` onsets via `env()`.

---

# 8. FLOW
```
generate() → buildPattern(seed) → rebuildEvents() → draw()
scheduler() fires events on the audio clock at loopStartCtx + ev.t
Set: pick selecta → startSet (or, while playing, stage via yoReroll + skipArmed)
   → build/condition loop → yoReroll stages next → scheduler swaps at the BAR boundary via applyPendingSwap
```

---

# 9. THE DESIGN SYSTEM  (governed — NOT Cortex)
<!-- @tags: DESIGN_SYSTEM -->
The exact UI/UX values. C §10 explains what it is and why it's governed; this is the identifier/value
index. Universal rules — change any = audit the whole UI.

**Design tokens (real CSS values)**
```
/* Section frame (Info/Controls/Voices/Mixer/Matrix panels + module boxes) */
background:  #0a0a0a
border:      1px solid #2a2a2a
radius:      6px (panels) · 5px (modules) · 4px (inner controls/buttons)
/* Inner control fill */
control-bg:      #151515
control-border:  1px solid #333
control-height:  30px   /* EVERY module control is 30px tall */
```

**Typography**
- Section headers (`.sec-hdr .txt`): 11px, weight 600, uppercase, letter-spacing 1px, Courier New (mono).
- Mono UI (values, counters, selectors, buttons): `'Courier New', monospace`.
- Display/body: `'SF Pro Display', -apple-system, BlinkMacSystemFont, sans-serif`.
- Module selector text: 11px mono, fully readable (flex-grows — never clip "909 Kick" to "909 Kic").

**Layout & spacing**
- Platform target: Chrome on MacBook Pro 14" — the whole TRACKS tab fits one screen.
- `.sec-hdr`: margin `6px 0 4px`, min-height 18px, cyan bar (3×13px) + cyan uppercase label + `#262626` rule.
- `.devstrip`: flex, gap 6px, padding 6px, margin-bottom 6px, wrap.
- `.devmod`: padding 6px, min-width 120px, flex:1; one 30px control row.
- `.dm-btn`: 34px × 30px, `#151515`/`#333`, radius 4px.

**Color semantics (each color MEANS something)**
| Color | Hex | Meaning / where |
|---|---|---|
| Cyan | `#00d4ff` | Selection/focus + ALL section headers; focus rings; active tab. |
| Magenta | `#ff006e` | Kill/mute state; matrix playhead. |
| Green | `#00e08a` | Audio-effect ON (reverb sends, master FX `fxon` glow). |
| Yellow/Orange | `#ffcc00`/`#ffa500` | Accent/time-slot warmth; per-voice lane colors. |
| Purple | `#c77dff` | Per-voice lane color (synth-ish lanes). |
| Red | `#e0114b` | Engaged kill (solid-red `.killed`). |

Voice-lane colors carry through matrix, modules, and Glover for one identity per voice.

**Component patterns**
- Section header = `.sec-hdr` → `.bar` (3px colored) + `.txt` (uppercase) + `.rule` (filler). Always cyan.
- Module box = `.devmod` in a `.devstrip`; one `.dm-row` (30px).
- Icon button = `.dm-btn` (+ `.dm-rand` dice / `.dm-fx` reverb ring / `.dm-kill` power). States: `:hover`
  tints to semantic color; `.fxon` green glow; `.killed` solid red.
- Value wheel = `.dm-wheel .wv` — scroll/drag numeric cell, 30px, centered.
- Matrix readout chips = `.time` (loop `#timeLoop`, session `#timeSess`); `#timeLoop.finishing` blink.
- Master-FX cluster (mixer header): `.vh-mlabel` + `.vh-mfx` + `.vh-mamt` + `.vh-mkill` (scoped to
  `#mixerHdr` — move the CSS scope if the cluster moves).

**Hard rules (anti-drift)**
1. Section headers are ALWAYS cyan `#00d4ff`. Never recolor a header.
2. Inherit existing classes; never hand-roll a parallel style.
3. No text labels on modules except Matrix lane chips.
4. Uniform 30px control height (selector, level, buttons) so rows align.
5. Buttons are square-ish (34×30), `#151515`/`#333` — not pills, not full-width.
6. When a styled element moves between headers, move its CSS scope too.
<!-- @/tags -->

---

# 10. AMU SEAM (Sozo / omsanalyze)
Explained in C §11. Yo DNA is derived from AMU JSON sidecars (`omsanalyze` → `MusicUnderstandingSession`):
bpm, beats/bars timestamps, key (time-ranges), loudness (LUFS), structure (sections/phrases/segments), and
— omsanalyze v2 — real `instrumentActivity` (bass/drum/other/vocal, activity curves + ranges). Per-band
DENSITY is AMU-derived and wired. Per-drum onset PLACEMENT not yet extracted (the MIDI-DNA layer).

---

# 11. CONSTRAINTS
Single-file, zero-dependency, offline, Chrome-targeted. No external fonts/frameworks/samples. All synthesis
generated. App is minor-locked. Bars are fixed dropdown values (never multiplied off-grid).
