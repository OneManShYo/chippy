# OMS Chippy [Beta V1_7_0] — SYSTEM REFERENCE

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
- `GENRE{}` — per-genre `{name, bpm, tempo:[min,max], build:fn}`. **House-only as of V1.7.0** (techno/
  breaks/dnb/bigroom builders REMOVED while the YoConditioner is dialed in on JNO; return rebuilt later).
- `SELECTAS[]` — selectas: `{id,name,genreChance,barsAllowed[],barsChange,pool[],bpm:[min,max], corpus[]?, profile?}`.
  Ships one: `jno` (Juice Night Out). `selecta()` = active; `selectaIdx`.
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

Applies ONLY under party mode (`partyOn`). No party ⇒ raw stock house (neutral gate, no moves).

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
**Wired to audio (V1.7.0):** `mode`, `swing`; per-band DENSITY (`setDensityFromProfile`→`dGate`→`gk`,
applied at the house builder's band probability points); `maxBpmJump` (clamp in `partyReroll`);
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
- Party: `startParty(mins)`, `partyReroll()` (incl. maxBpmJump clamp), `stopParty()`, `selecta()`, `vibe()`.
- Render: `draw()` (matrix), `drawFace()`, `drawRave()` (Glover), `buildStrip()` (voice modules).

## FLOW
generate() → buildPattern(seed) → rebuildEvents() → draw() ; scheduler() fires events on the audio
clock. Party: startParty → build first loop (conditioned by the selecta profile) → partyReroll stages
next → scheduler applies at boundary via applyPendingSwap.

## PARTY GRID OWNERSHIP
- SELECTA owns bars (barsAllowed+barsChange) + crate (pool+genreChance).
- GENRE owns tempo (GENRE[].tempo, the hard clamp). Selecta bpm biases within it.
- TIME SLOT owns level/density/pace (VIBES).
- PROFILE (Yo DNA) conditions composition (mode/swing/density…) + mixing (maxBpmJump/moves…) in party.

## CONTROL-BEHAVIOR CATEGORIES
RESTART (genre, selecta → quantizeKey, next beat) · CONTINUE (tempo, swing, voices → live) ·
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
