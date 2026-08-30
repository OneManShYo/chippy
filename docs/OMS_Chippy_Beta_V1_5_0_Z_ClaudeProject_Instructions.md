# OMS Chippy [Beta V1_5_0] — CLAUDE PROJECT INSTRUCTIONS

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

## VERSIONING (three-digit: major.minor.iteration)
- **One change per iteration; increment only; never overwrite an iteration; never invent version numbers.**
- Iterations bump the third digit during a work session (V1_5_01, _02, …).
- A **published release bumps the MINOR** and resets iteration to 0 (V1.3.0 → V1.5.0).
- Build/doc FILENAMES use UNDERSCORES with the version expanded (`OMS_Chippy_Beta_V1_5_0.html`).
  Folders use dotted form (`V1.5.0_Deliverables/`). ONE separator per project — OMS = underscores.
- The full release + git workflow lives in the OMS Suite VersionWorkflow doc (currently V2.4). Follow
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
Define `{id,name,genreChance,barsAllowed,barsChange,pool,bpm}` in DJ_MODES. Optionally add a
`profile:{mode,swing,structure,dynamics}` derived from AMU analysis of reference tracks (see the AMU
pipeline). Bars must be real dropdown values. Genre owns tempo; selecta biases within it.

## ADDING A GENRE
Add to `GENRE{}` with `{name,bpm,tempo:[min,max],build:fn}` and write the `build(P,R)` pattern
function (push events into P.drums/perc/bass/lead/synth/accent/fills/fx). Add to any selecta `pool`
that should draw from it.

## THE AMU → PROFILE PIPELINE
Selecta profiles come from Sozo's `omsanalyze` (Apple Music Understanding) JSON sidecars of real
tracks. Extract the common signature (mode, swing, BPM, structure grid, dynamics), encode as a
profile. It's learning the DNA, not sampling. See C-doc §9 and Y-doc AMU seam.

## TERMINOLOGY (OMS + Chippy vocabulary)
- **Cortex / Node / Leaf** — system stability tiers (Cortex = core, don't casually touch).
- **Selecta** — the DJ/style (record crate). **Time slot** — the club-night arc (Opener…Closer).
- **Party grid** — the automation constraint matrix. **Profile** — a selecta's AMU-derived DNA.
- **Glover** — the light-show tab (from "gloving"). **Voices/modules** — the 8 lanes.
- **FX (voice 8)** = the risers/glitches/horn effects voice. **Master FX** = the reverb bus.

## WELL-KNOWN GOTCHAS (from real incidents)
- Beat-quantize must anchor to a fixed origin (playOriginCtx), or targets recede past the horizon.
- The mobile responsive layer caused a black screen (V1_0_32) — build mobile fresh, one change,
  check DevTools console. (Parked.)
- Git identity can be "set but wrong" (a literal placeholder) — verify the VALUE is
  wes@itswessmithyo.com before any commit.
