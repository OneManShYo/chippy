# OMS Chippy [Beta V1_6_0] — SYSTEM REFERENCE

Machine-readable reference for an AI/dev picking up the single-file codebase. Names below are the
actual identifiers in the HTML's inline script.

## CONSTANTS / THEME
- `CY #00d4ff, MAG #ff006e, YEL #ffcc00, GRN #00e08a, PUR #c77dff, ORG #ffa500` — palette (voices/UI/Glover).
- `STEPS_PER_BAR = 16` (16th grid). `BARS` loop length; `TOTAL_STEPS = STEPS_PER_BAR*BARS`; `setBars(n)`.
- `LOOKAHEAD = 0.25` (scheduler lookahead seconds).
- `A4 = 440`; `midiToHz(m)`; `NOTE{}`, `MINOR[]` (natural minor); `keyTonic` (default "A", app is minor-locked).

## AUDIO NODES
- `actx` AudioContext; `master` master gain → destination.
- `noiseBuf` (white), `pinkBuf`, `brownBuf`.
- `reverbBus` (ConvolverNode, generated impulse) → `reverbWet` → `master`.
- `curDest` — per-fire routing target (master, or a tap that also sends to reverbBus).

## DATA CONTAINERS
- `VOICES[]` — 8 voices `{id,name,color,osc,on,lvl,decay,...}`: drums, perc, bass, lead, synth, accent, fills, fx.
  `V{}` = id→voice. Voice ids also index pattern arrays `P{}`.
- `GENRE{}` — per-genre `{name, bpm, tempo:[min,max], build:fn}`: bigroom, house, techno, breaks, dnb.
- `DJ_MODES[]` — selectas: `{id,name,genreChance,barsAllowed[],barsChange,pool[],bpm:[min,max], profile?}`.
  Order: house, battle, open, warehouse, jungle, giorgio.
- `VIBES[]` — time slots: `{id,name,barBias,density,level}`: opener, support, headliner, closer.
- Selecta `profile:{ mode, swing:[min,max], structure:[phrase,segment,section], dynamics }` (§ Party Grid).
- `OSC_OPTS`, `KICK_OPTS`, `FX_OPTS` — module dropdown option lists.
- Title banks: `BRANDS`, `MONIKERS`, `VENUES` (Wes-contributed), `FX_DESC`, `FX_END` (auto flavor).

## EVENT / PATTERN SHAPES
- Pattern `P = {drums:[],perc:[],bass:[],lead:[],synth:[],accent:[],fills:[],fx:[]}`.
- Event `{step, dur, note|null, accent?, snare?, crash?}`. `note` = MIDI or null (percussive).
- `rebuildEvents()` flattens P into a time-sorted `events[]` (seconds-into-loop `.t`).

## KEY FUNCTIONS
- Synthesis: `playKick`, `playNoise`, `playPulse/playTone` (osc voices), `playSweep`, `playGlitch`, `playHorn`.
- Routing: `fireVoice(ev,when)` — sets curDest (per-voice/master reverb), dispatches to the right synth.
- Generation: `buildPattern(seed)` → calls `GENRE[genre].build(P,R)`; `makeTitle()`.
- Genre builders: `house, techno, breaks, dnb, bigroom` (each writes P). `mkRnd(seed)` seeded RNG.
- Clock: `scheduler()`, `nextBeatCtx()`, `quantize()/quantizeKey()`, `applyPendingSwap()`.
- Party: `startParty(mins)`, `partyReroll()`, `stopParty()`, `djMode()`, `vibe()`.
- Render: `draw()` (matrix), `drawFace()`, `drawRave()` (Glover), `buildStrip()` (voice modules).

## FLOW
generate() → buildPattern(seed) → rebuildEvents() → draw() ; scheduler() fires events on the audio
clock. Party: startParty → build first loop (pool/bars-correct) → partyReroll stages next →
scheduler applies at boundary via applyPendingSwap.

## PARTY GRID OWNERSHIP
- SELECTA owns bars (barsAllowed+barsChange) + crate (pool+genreChance).
- GENRE owns tempo (GENRE[].tempo, the hard clamp). Selecta bpm biases within it.
- TIME SLOT owns level/density/pace (VIBES).
- PROFILE (optional, AMU-derived) adds mode/swing/structure/dynamics DNA.

## CONTROL-BEHAVIOR CATEGORIES
RESTART (genre, selecta → quantizeKey, next beat) · CONTINUE (tempo, swing, voices → live) ·
WAIT-FOR-LOOP (length → pendingLen, loop boundary).

## AMU SEAM (Sozo / omsanalyze)
Selecta profiles are derived from AMU JSON sidecars (omsanalyze → MusicUnderstandingSession). JSON gives
bpm, beats/bars timestamps, key(time-ranges), loudness(LUFS), structure(sections/phrases/segments).
instrumentActivity (per-instrument onsets) is stubbed → per-drum placement not yet extracted.

## CONSTRAINTS
Single-file, zero-dependency, offline, Chrome-targeted. No external fonts/frameworks/samples. All
synthesis generated. App is minor-locked. bars are fixed dropdown values (never multiplied off-grid).
