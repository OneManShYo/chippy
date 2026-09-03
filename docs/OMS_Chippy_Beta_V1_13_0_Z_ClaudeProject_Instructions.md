# OMS Chippy [Beta V1_13_0] — CLAUDE PROJECT INSTRUCTIONS

How an AI (or dev) should work in the Chippy project. Read this first. These rules were learned the
hard way; follow them.

## WHAT CHIPPY IS
A single-file, browser-native, generative chiptune dance-music instrument + light show. Generation,
not playback. Part of the OMS suite with Dojo (performance instrument) and Sozo (AMU analysis app).
Chippy was seeded from the Dojo deliverable structure and shares OMS vocabulary (Cortex/Node/Leaf,
single-file philosophy, master clock, module system).

## CORE PRINCIPLES (do not violate)
- **Single file, zero dependencies.** No external fonts, frameworks, sample libraries, or network
  calls. Must run offline from one HTML file. This is non-negotiable — it's the app's identity.
- **Generation, not playback.** No recordings/samples as content. Everything is seed + rules.
- **Read the code before acting.** No guessing at how something works — open it and look.
- **Terminal commands only, one at a time**, pasted back before the next. Never describe filesystem
  ops without a runnable command. (Wes's standing workflow preference.)
- **Only add what's asked.** Do not invent content (e.g. word-bank terms). Contributed lists are
  provenance-classed — only Wes's terms go in his classes; anything Claude adds is labeled separately.
- **No opinions injected into his app.** It's his creative work; match his intent, don't editorialize.
- **Follow the Design System (Cortex).** Before building or moving any UI, read the Design System in the
  Y System Reference and inherit its tokens/classes (`.sec-hdr`, `.devmod`, `.dm-btn`, the palette). Do
  NOT hand-roll parallel styles. Hard rules: section headers are always cyan `#00d4ff`; module controls
  are a uniform 30px height; no text labels on modules except the Matrix lane chips; when a styled
  element moves between containers, move its CSS scope too. (Skipping this is how UI drift happens.)

## THE MESSAGE SYSTEM (Cortex — the ONLY feedback channel)
All transient user feedback goes through `showMessage(text, type, onClick)` → the footer `#messageBar`.
Ported from Dojo; hover-driven. Types + colors are fixed: `info` cyan (hover help), `state` orange
(value changed), `error` magenta (failure). NEVER hard-code a user-facing string at a call site: every
message lives in the central `MSG` registry (`MSG.hint` / `MSG.state` / `MSG.error`) — one place to
change any message. No native `title=` tooltips. The C-doc field/label table is generated from `MSG`;
keep them in sync.

## VERSIONING (three-digit: major.minor.iteration)
- **One change per iteration; increment only; never overwrite an iteration; never invent version numbers.**
- Iterations bump the third digit during a work session (V1_7_01, _02, …).
- A **published release bumps the MINOR** and resets iteration to 0 (V1.3.0 → V1.8.0).
- Build/doc FILENAMES use UNDERSCORES with the version expanded (`OMS_Chippy_Beta_V1_7_0.html`).
  Folders use dotted form (`V1.8.0_Deliverables/`). ONE separator per project — OMS = underscores.
- The full release + git workflow lives in the OMS Suite VersionWorkflow doc (the OMS Suite PM Packaging Workflow — follow its current version). Follow
  its numbered steps and the HUMAN PAUSE / GATE REVIEW before any push.

## BUILD PROTOCOL (per iteration)
1. Read the current file. 2. Make ONE change. 3. Stamp the new iteration number. 4. Syntax-check the
JS (`node --check` on the extracted script). 5. Stage + present the file. Never stack unrelated changes
onto one iteration — it destroys rollback points.

## THE CHANGELOG vs BACKLOG
- **Backlog** (`OMS_Chippy_PM_Backlog...md`) = the firehose: every BLOG# ticket, OPEN/PARKED/CLOSED.
  Closed items truncate on each build (preserved in prior bundles).
- **Changelog** (B doc) = curated, release-specific "what shipped" — written fresh each release from
  the closed BLOGs, NOT just version-bumped. Reviewed at the gate.

## ADDING A SELECTA (party grid)
Define `{id,name,genreChance,barsAllowed,barsChange,pool,bpm}` in SELECTAS. Optionally add a
`profile:{}` (the Yo DNA — see THE YOCONDITIONER below) plus `corpus:[]` (the reference tracks it was
derived from). Bars must be real dropdown values. Genre owns tempo; selecta biases within it.

## ADDING A GENRE
Add to `GENRE{}` with `{name,bpm,tempo:[min,max],build:fn}` and write the `build(P,R)` pattern
function (push events into P.drums/perc/bass/lead/synth/accent/fills/fx). Gate band fire-probabilities
with `gk('band',p)` so density conditioning applies. Add to any selecta `pool` that should draw from it.
(NOTE: V1.8.0 is HOUSE-ONLY — non-house builders were removed; reintroduce them rebuilt to the
YoConditioner standard.)

## THE YOCONDITIONER (Cortex — how the music gets generated)
The conditioned generative layer, above the swappable sound set. TWO PRIMITIVES, KEPT DISTINCT:
- **Yo DNA** = the NOUN. The committed reference fingerprint (a selecta's `profile:{}`). Source is
  anything Wes commits: AMU (omsanalyze), MIDI, DJ-mixes, or authored knowledge — not just AMU.
- **Yo Conditioning** = the VERB. Applying that DNA to bias the generator. Corpus conditioning (the ML
  term) as a pun. NOT training, NOT retrieval — it tilts a stochastic process, never writes the output.
Two roles of DNA: COMPOSITION (what the music is — mode/swing/density/structure…, mostly AMU) and
MIXING (what the DJ does — maxBpmJump/transition/moves…, authored). Applies ONLY in party mode.
When wiring a new DNA field: read it from `profile`, bias a real generator parameter, and record in the
C-doc §2.5 whether it's WIRED or CAPTURED. Keep the noun/verb separation crisp — it's what makes the
method teachable. (Name "YoConditioner" is a working placeholder; may change.)

## THE AMU → PROFILE PIPELINE
Yo DNA is derived from Sozo's `omsanalyze` (Apple Music Understanding) JSON sidecars of real tracks.
Extract the common signature (mode, swing, BPM, loudness, structure, and per-band densities from v2
`instrumentActivity`), encode as a `profile:{}`. It's learning the DNA, not sampling. omsanalyze **v2
emits real instrumentActivity** (bass/drum/other/vocal) — density is AMU-derived; per-drum note
placement is not (MIDI-DNA layer). See C-doc §9 and Y-doc AMU seam.

## TERMINOLOGY (OMS + Chippy vocabulary)
- **Cortex / Node / Leaf** — system stability tiers (Cortex = core, don't casually touch).
- **YoConditioner** — the conditioned generative layer (Cortex). **Yo DNA** (noun) = committed
  fingerprint; **Yo Conditioning** (verb) = biasing generation with it.
- **Selecta** — the DJ/style (record crate). **Time slot** — the club-night arc (Opener…Closer).
- **Party grid** — the automation constraint matrix. **Profile** — a selecta's Yo DNA.
- **Glover** — the light-show tab (from "gloving"). **Voices/modules** — the 8 lanes.
- **FX (voice 8)** = the risers/glitches/horn effects voice (a SOUND, in the matrix). **Audio effects** =
  the master mixer processing (reverb/echo/filter/backspin + EQ/comp/limiter/volume). NEVER call the audio
  effects "FX" — FX is the voice; audio effects are processing. Internal naming is `audioEffects*`/`master*`,
  never `fx`.

## WELL-KNOWN GOTCHAS (from real incidents)
- Beat-quantize must anchor to a fixed origin (playOriginCtx), or targets recede past the horizon.
- The mobile responsive layer caused a black screen (V1_0_32) — build mobile fresh, one change,
  check DevTools console. (Parked.)
- Git identity can be "set but wrong" (a literal placeholder) — verify the VALUE is
  wes@itswessmithyo.com before any commit.
