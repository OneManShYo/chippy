# OMS Chippy — Changelog

## V1_3_0 — Performance & Recording Release (2026-08-29)

Minor-version roll: closes out the V1.0 iteration line (V1.0.1 → V1.0.31) as a substantial feature release. Keyboard control, recording, and the transport readout were all overhauled.

### Added
- **WASD grid navigation** across the Controls and Voices rows: A/D move within a row (wrapping continuously across both rows), W/S jump between rows landing at the left. Works from wherever focus is; a cold press lands on the top-left control. Tab is left untouched.
- **Arrow-key control** on the voice-type dropdowns (and Controls dropdowns): left/right/up/down cycle options live via the shared bindUctl binding.
- **Ableton-style session counter** in the MATRIX header: continuous bars.beats.sixteenths since play (1-based, never wraps), alongside the loop-relative bar/beat. Rests at 1.1.1 when stopped.
- **Persistent record cluster**: persistence, orientation, record, and readout are always visible (not gated to the RAVE tab); recording no longer force-switches you to that tab.

### Fixed
- **Long recordings now play back correctly.** Root causes: the recorder buffered the whole clip as one blob (fixed with a 1s timeslice), and a bare `video/mp4` request made Chrome emit VP9/Opus inside a fragmented MP4 that QuickTime rejected. Now only genuine H.264/AAC is accepted as MP4 (iOS Safari), otherwise clean WebM; on capable desktop Chrome this yields a real, QuickTime-playable MP4.
- Record readout keeps a fixed `00:00` format and reserved slot so hitting record never shifts the layout.

### Changed
- **Control-behavior model formalized into three categories** (documented in-code): RESTART (genre, DJ — quantize to the next beat, clean loop restart), CONTINUE (tempo, swing, voices — apply live, no restart), and WAIT-FOR-LOOP (length — finish the current loop, then switch at the boundary). Tempo and swing were moved off beat-quantize to live application; length now defers to the loop end.
- Record readout border is dim at rest and brightens when recording, matching the other controls.
- **Removed the WAV export button** — the record button covers capture now.

---

## V1_0_14 — Beat-Quantize Clean Cut (2026-08-29)

### Fixed
- **Control changes while playing now land cleanly on the next beat with no audible skip.** The beat-quantize swap is computed against a fixed playback-origin grid and an absolute audio-clock target, so the boundary can no longer recede past the scheduler's look-ahead window (the bug that made changes fail to apply in the V1_0_11/12 attempts). Any already-scheduled notes from the outgoing pattern finish; nothing new from it is scheduled past the cut, so the loop restarts glitch-free on the beat.

### Changed
- Beat-quantize now shares the same clean-cut principle as the skip/party swap, aligned to the next beat instead of the next bar.

---

## V1_0_1 — Persistence Button Relabel (2026-08-29)

### Changed
- **Renamed the ∞ button's function from "continuous play" to "persistence."** It keeps playback alive when you switch to another browser tab — not a loop/continuous-play control. Corrected the tooltip and on/off status messages (mechanism unchanged).

*First iteration on the V1.0 release series (V1.0.0 → V1.0.1). The V0_0_x entries below are the pre-release development history.*

---

## V0_0_54 — Tight Button Cluster (2026-08-29)

### Fixed
- **The rave controls now sit together as a tight cluster.** The empty record timer was reserving a fixed width between REC and ∞, forcing a gap; it now collapses to zero when empty and only appears (to the left of the buttons) while actually recording. Orientation, REC, and ∞ now group cleanly at the far right.

---

## V0_0_54 — Overlapped Orientation Icon (2026-08-29)

### Changed
- **The orientation icon's landscape and portrait marks now overlap concentrically** (Adobe-style) inside the same square button instead of sitting side by side — the button keeps its normal size, and the active orientation is highlighted over the dimmed other.

---

## V0_0_54 — Coloured Control Icons (2026-08-29)

### Changed
- **REC is now always red** (its identity as a record button) and **blinks only while actually recording**. The timer is red too.
- **Orientation toggle is cyan**, continuous-play (∞) is green — each control now carries its own colour for quick recognition.

---

## V0_0_54 — Orientation Icon Shows Both States (2026-08-29)

### Changed
- **The orientation toggle now shows both a landscape and a portrait mark in the same button**, with the active one bright and the other dimmed — so it clearly reads as "switch orientation" and indicates which you're on.

---

## V0_0_54 — Recordings Now Have Audio (2026-08-29)

### Fixed
- **The rave recording now includes sound.** It was capturing the canvas only, so the exported video played silent. It now taps the live Chippy master audio bus and mixes it into the recording as an audio track, so the file has picture *and* sound.
- Recording codecs updated to carry audio — MP4/AAC on iOS, WebM/Opus on desktop Chrome — at 192 kbps. Whatever's playing while you record is what you'll hear back.

---

## V0_0_54 — Modular Icon Buttons (2026-08-29)

### Changed
- **The three top-bar controls are now identical modular icon buttons** — same fixed 30×30 size, border, radius, and style. Icons only: orientation (▬ landscape / ▮ portrait), record (⏺ / ⏹ stop), and continuous play (∞).
- **∞ is now last in the group**, right-aligned, so it always sits in the exact same far-right spot; the rave controls appear to its left and never shift its position.

---

## V0_0_54 — Contextual Rave Controls in the Global Bar (2026-08-29)

### Changed
- **Reorganised the top-right controls.** The ∞ (continuous play) toggle is now first and always visible. The **16:9/9:16 toggle and REC button live in the same global bar but only appear when you're on the RAVE ON tab** — contextual, clean, and off the canvas.
- Removed the floating aspect button that used to sit on the canvas.

---

## V0_0_54 — Shareable Recording Format (2026-08-29)

### Changed
- **The recorder now grabs the most shareable format the browser natively supports.** It tries MP4/H.264 first and falls back to WebM only if MP4 isn't available. In practice: **on iPhone (iOS Safari) you get a real .mp4** that drops straight into the camera roll and posts to socials with no conversion; on desktop Chrome it's still WebM (Chrome can't natively record MP4). The file is named with the correct extension for whatever actually recorded, and the confirmation says which format you got.
- No new dependencies — stays single-file. The phone flow (pick ratio → REC → stop → camera roll → post) works with zero extra steps once you're on iOS.

---

## V0_0_54 — Rave Controls Relocated (2026-08-29)

### Changed
- **Fixed the RAVE ON layout** — the control row above the canvas was pushing everything down (and wasting a whole row in 9:16). Removed it.
- **Moved REC up to the tab bar** next to the ∞ toggle, so it's always visible and clickable from anywhere. Hitting REC auto-switches to the RAVE tab (so it always captures live visuals) and shows a running timer; click again to stop & download.
- **Moved the 16:9 / 9:16 toggle to a floating button in the top-right corner of the rave canvas** — an unobtrusive glassy overlay that takes no layout space.

---

## V0_0_54 — Rave Video Export + Orientation Toggle (2026-08-29)

### Added
- **Record button on RAVE ON** — captures the rave canvas to a downloadable **.webm video** (VP9, 60fps, 12 Mbps) with a live timer. Hit REC, let it rave, hit STOP → `ChippyRave_1920x1080_HHMMSS.webm`. Single-file, no dependencies; drops straight into an FFmpeg/ProRes → YouTube pipeline.
- **Orientation toggle (16:9 ⇄ 9:16)** — flip between landscape (YouTube) and vertical (Shorts / TikTok / Reels) with one button; the scene re-lays-out to fit.

### Changed
- **The rave canvas now renders at a locked, true resolution** — 1920×1080 landscape or 1080×1920 vertical — so recordings (and screen captures) are genuine 1080p regardless of window size, instead of varying with the display.

---

## V0_0_54 — Rave Canvas 16:9 (2026-08-29)

### Changed
- **RAVE ON canvas is now a standard 16:9 (YouTube) ratio** instead of a variable 70vh box — taller and consistent, capped at 78vh so it never overflows.
- **Its border now matches the main section frames** (1px #2a2a2a, 6px radius; was 8px).

---

## V0_0_54 — Logo Link Hardening (2026-08-29)

### Changed
- **Made the YoBot logo link fully explicit** — `https://onemanshyo.com/` (trailing slash) with `rel="noopener noreferrer"`. This rules out the app as the cause of the itswessmithyo.com redirect some users saw; that behaviour is browser-cached (HSTS / cached 301) or host-level domain forwarding, not anything in the file. (Test in Incognito, and clear the cached policy at chrome://net-internals/#hsts for onemanshyo.com.)

---

## V0_0_54 — Consistent Section Frames (2026-08-29)

### Changed
- **All four main section frames (Info, Controls, Voices, MATRIX) now share the same outer module border** — 1px #2a2a2a, 6px radius, #0a0a0a fill. Voices and MATRIX previously had no outer frame; now every section reads as the same big framed module for a consistent look.

---

## V0_0_54 — Full Tooltip Coverage (2026-08-29)

### Added
- **Hover tooltips across the whole UI.** Filled the gaps so everything explains itself: the genre and length selectors, the party button, and the MATRIX roll now have tooltips, joining the tabs, transport buttons, tempo/swing/DJ wheels, party duration, and the per-voice cards. Hover anything to see what it does.

---

## V0_0_54 — Clickable Logo (2026-08-29)

### Added
- **Clicking the YoBot logo mark opens onemanshyo.com** in a new tab, with a pointer cursor and a subtle hover lift.

---

## V0_0_54 — Smaller Logo Mark (2026-08-29)

### Changed
- **YoBot logo mark in the header shrunk ~30%** (34px → 24px) for a subtler top-right mark.

---

## V0_0_54 — Papa Chippy & The Kids (2026-08-29)

### Added
- **RAVE ON now has Papa Chippy's kids** — six colored, ~half-size Chippys orbiting Papa in the center, each on its own radius, speed, and direction, bouncing on the kick and drawn in the rave palette. Slightly squashed orbits give a little depth so they dance around him. Papa stays the big yellow centerpiece.

### Internal
- `drawChippyAt` now takes an optional head colour (defaults to Chippy yellow) so the kids can come out in colours.

---

## V0_0_54 — Chippy Back on the Card, YoBot as Logo Mark (2026-08-29)

### Changed
- **Reverted the Info card to the single dynamic Chippy** — the fun, animated face (drifting eyes, X-out on the beat, morphing mouth) is back as the centerpiece.
- **Moved the YoBot logo up to the top-right of the header** as a proper logo mark, next to "Hi Friend, Let's Dance". Still embedded base64, aspect-correct, single-file.

---

## V0_0_54 — YoBot Un-Squashed + Smileys Restored (2026-08-29)

### Fixed
- **YoBot no longer looks squashed.** The Info-card canvas had a fixed 480×300 backing store stretched to a wide, short display box, distorting everything drawn on it. The canvas now sizes its backing store to its actual displayed pixels (× DPR) on load, resize, and tab-switch, so the logo renders at its true aspect ratio.
- **Logo now drawn at ~50% size**, aspect-ratio preserved.
- **Floating smileys restored to full strength** — same look as the classic Chippy face (solid, proper eyes, drift + X-out on the beat) instead of the faded, subdued ghosts. They missed their father Luke. ;)

---

## V0_0_54 — Next / Skip Button (2026-08-29)

### Added
- **Next button (⏭)** next to play, plus the **N** key. If the current loop isn't your mood, skip ahead:
  - **In party** — jumps to the next DJ pick (stages a reroll and cuts to it).
  - **Playing, not party** — swaps in a fresh roll of the current genre.
  - **Stopped** — just generates a new one.
- The skip is **bar-quantised** — it lands on the next bar's downbeat, so it's musical and seamless rather than a jarring mid-bar cut.

---

## V0_0_54 — YoBot Logo on the Info Card (2026-08-29)

### Added
- **The Info-card smiley is now the YoBot logo** (SMPTE-bars headphones mascot), embedded as base64 so the build stays single-file (adds ~32 KB; total ~112 KB). It pulses on the kick with a gentle bob.
- **A few faint Chippy smileys now drift slowly behind the logo** for depth, with a tiny beat bob. (The animated smiley still lives in the RAVE ON scene and the overview strip.)

### Notes
- Logo was auto-cropped from the transparent 1080×1920 source and downscaled to 300 px for a small footprint.

---

## V0_0_54 — Blue Labels + MATRIX Alignment (2026-08-29)

### Changed
- **Section labels (Info, Controls, Voices, MATRIX) recoloured to the tab blue** so they match the tab labels.
- **Aligned the MATRIX overview strip and roll to the same width as the Controls/Voices rows** — the canvas modules were inset ~10px on each side; now they sit near-flush with the rows above for a tighter, justified layout.

---

## V0_0_54 — MATRIX (2026-08-29)

### Changed
- **Renamed the "GRID" section to "MATRIX".**

---

## V0_0_54 — Label Tidy (2026-08-29)

### Changed
- **All section labels (Info, Controls, Voices, GRID) recoloured to the Chippy orange** for a consistent look.
- **Renamed the "Piano Roll" section to "GRID".**

---

## V0_0_54 — Kick Flavors (2026-08-29)

### Changed
- **The Drums voice is now a kick-flavor selector instead of an oscillator selector.** Switching it always keeps it a kick (it's the anchor of the whole groove) — you now pick between drum-machine-style kicks live: **909** (snappy analog), **808** (deep booming sub, long tail), **Punch** (hard, mid-forward), **Click** (tight techno click), and **Noise** (raw noise-forward). Defaults to 909.
- Every other voice keeps its full oscillator dropdown (pulse/triangle/noise) — switching those to reshape the sound live is unchanged, since that's the fun.

### Why
- Previously the Drums voice shared the generic oscillator list, so switching it to Triangle/Pulse turned the kick into a pitched instrument and collapsed the beat. Now the kick stays a kick, and you get variety *within* the kick family — like swapping drum machines under a running groove.

---

## V0_0_54 — Techno 135 (2026-08-29)

### Changed
- **Techno tempo bumped to 135 BPM** (from 132).

---

## V0_0_54 — Techno + House/Rave DJs (2026-08-29)

### Added
- **Techno genre** — a new four-on-the-floor style at 132 BPM: driving offbeat hats, a hypnotic rolling 16th sub-bass, a repetitive minor stab riff, sparse high blips. Minimal and relentless. Added to the genre dropdown (after House).
- **Two new DJ modes**, bringing the roster to five (House · Battle · Open Format · Warehouse · Rave):
  - **House** — plays **House only**; now the **default** DJ mode.
  - **Rave** — same as Warehouse (FOTF: house / techno / big room) but **+20% on everything** — tempo ×1.2 and loops turn over ~20% faster.

### Changed
- **Warehouse's FOTF pool now includes Techno** (house / techno / big room), so it can ride techno as well as house all night.
- A mode whose current genre falls outside its pool now snaps into the pool on the next loop, so each DJ's identity holds immediately after you switch to it.

---

## V0_0_54 — Continuous-Play Icon in Tab Bar (2026-08-29)

### Changed
- **Moved the continuous-play toggle back up to the tab header row** so it's always visible, as an **infinity (∞) icon** instead of text. Green glow = on (default). Tooltip explains it.
- **Removed the hint text and the toggle bar from the RAVE ON tab** — the rave visual is now clean, just the full canvas.

---

## V0_0_54 — Cleaner Voice Cards (2026-08-29)

### Changed
- **Removed the coloured dots from the voice cards.** The colour cue now lives on the **level value** itself (each voice's number is tinted its colour), so it still reads as a legend without the extra dot. Tooltips still explain each voice. **Clicking anywhere on a card mutes/unmutes** it (the dropdown and level wheel still work normally); a muted card dims and its value greys out.

---

## V0_0_54 — RAVE ON tab + Background Default (2026-08-29)

### Added
- **RAVE ON** — a new second tab (TRACKS · RAVE ON · ABOUT). A full-screen, beat-reactive light show: colour-strobing background that flashes on the kick, rotating radial light beams, concentric rings pulsing out on the beat, a scatter of golden-angle dancing dots reacting to snare/lead, and **Big Chippy raving in the centre** (bouncing on the kick, eyes X-ing out on the downbeat, mouth morphing) — with a white burst on the crash. Reacts to whatever's playing, party or manual.
- **All three tabs now have hover tooltips** explaining what each does.

### Changed
- **Background play ("keep playing in other tabs") is now ON by default**, and its toggle **moved into the RAVE ON tab** (under the visual) instead of sitting alone in the tab bar — labelled in plain language so it's clear what it does.

### Notes
- The rave visual runs off the same beat-detection that drives Chippy's face, so it needs no extra analysis.

---

## V0_0_54 — Warehouse = Four-on-the-Floor (2026-08-29)

### Changed
- **Warehouse mode now only moves between four-on-the-floor styles** (House and Big Room / EDM) — it never breaks into Breaks or D&B, so the steady kick-on-every-beat pulse holds all night. Each DJ mode now has an explicit genre pool (Battle and Open Format keep the full spread; Warehouse is FOTF-only).

---

## V0_0_54 — Compact Controls (2026-08-29)

### Changed
- **Generate and Play/Pause are now icon-only** (⚄ and ▶/❚❚) with hover tooltips, freeing space so the controls row stays on one line instead of wrapping the DJ selector down.

---

## V0_0_54 — Background Play: Moved & Hardened (2026-08-29)

### Changed
- **Moved the "bg" toggle to the top-right of the tab bar** (by TRACKS / ABOUT), out of the controls row so it no longer causes the row to wrap. It's a more general/global control anyway.
- **Hardened background playback** so it actually keeps going on tab switch: the silent AudioWorklet pump now fires ~every 60 ms, with a plain-interval backup, an AudioContext `resume()` on each background tick, and a wider 250 ms scheduler lookahead so any pump stutter can't drop notes.

### Note
- Visuals still pause in a hidden tab (canvas can't draw when the tab isn't rendered) and resume on return; the audio is what keeps playing.

---

## V0_0_54 — Background Play (2026-08-29)

### Added
- **"bg" toggle** (next to the DJ selector) — keeps the music playing when you switch to another Chrome tab. Off by default; click to enable (turns solid/cyan).
- Chrome freezes a tab's `requestAnimationFrame` and throttles timers when it's in the background, which normally stalls the scheduler. This adds a **silent AudioWorklet** that runs on the (non-throttled) audio thread and pumps the scheduler ~every 80 ms while the tab is hidden, so the party keeps rocking while you work elsewhere. Inlined via `URL.createObjectURL` to preserve the single-file build.

### Notes
- Visuals (roll, Chippy) still pause in a background tab — canvas drawing is tied to `requestAnimationFrame` and can't run hidden — but they snap back when you return. Only the audio needs to keep going, and it does.
- Internal: the frame loop was split into `tick()` (scheduling/timing, can run headless) and drawing.

---

## V0_0_54 — DJ Styles (2026-08-29)

### Added
- **DJ style selector** — a wheel next to party + duration (scroll/drag/arrows) that sets how the party evolves:
  - **Battle** — frantic: hops genre ~90% of loops, short 4–8 bar loops. Cutting, everything-moving.
  - **Open Format** — the ~1-min hook-and-go feel: ~50% genre hops, 16–64 bar loops. (Default.)
  - **Warehouse** — hypnotic: rarely changes genre (~8%), stays mostly house, long 64–128 bar loops. "One song forever."
- Each mode drives genre-change frequency, the loop-length pool, and (for Warehouse) a house bias.

### Fixed / confirmed
- **The genre and length dropdowns now update live during a party** to show what's actually playing as it wanders — so anyone watching can see the current genre and loop length, not a stale "House."

---

## V0_0_54 — Longer Loops + Phrase Accents/Fills (2026-08-29)

### Added
- **Length dropdown goes up to 128 bars** (4 / 8 / 16 / 32 / 64 / 128). At ~120 BPM a 128-bar loop is ~1 minute; faster genres (D&B ~174) compress toward ~35 s. This centres loops around the open-format / short-attention ~1-minute phrase length — hear one structure and the hook, then move on.
- **Accents & Fills are now reliably present and phrase-structured.** They were almost never placed by the genre builders, so those lanes sat empty and long loops got boring. A new `phraseAccents` pass marks the phrasing: a small tick/fill at every 4-bar turnaround, a bigger fill + accent stab at every 8, and the biggest rolling fill + double stab at every 16 and the loop-end. It scales with length — a 4-bar loop gets an end tick, a 128-bar loop builds through the full hierarchy — so the long loops have motion and direction.

### Party
- Party bar-length weighting shifted toward the longer, ~1-minute loops (16–128) to match the open-format feel.

---

## V0_0_54 — Mouth Fix (2026-08-29)

### Changed
- **Toned down Big Chippy's mouth** — it was too wide and curved up far enough to crowd the eyes (looked a bit unhinged). Now smaller (narrower), thicker stroke, sitting lower on the face with a gentler curve, so it reads as a friendly grin that clears the eyes.

---

## V0_0_54 — Party: Seamless & Actually Switches Genre (2026-08-29)

### Fixed
- **No more skip/gap between party loops.** The old reroll reset the audio clock to `now + 0.06` at each boundary, which broke continuity. Now the next loop is *prepared ahead of time* (pattern, genre, bars, tempo all built into a pending buffer) and the scheduler swaps it in **exactly at the loop boundary** — `loopStartCtx += L`, no clock reset. Verified: 0.000000 s gap between loops. Fully continuous.
- **Party now actually changes genre.** It hops to a genre every ~1 in 2 loops and a guard guarantees it picks a *different* one (never re-picks the current), so it audibly moves through House / Breaks / D&B / Big Room instead of staying stuck. Bar length still varies ~1 in 4 loops. The roll, genre/length dropdowns, tempo, and info all switch together at the boundary.

### Internal
- Added `buildEventsFrom(pattern)` (builds an event list without touching the live `events`), and a `pendingParty` swap applied inside the scheduler at the boundary — the seam that makes it gapless.

---

## V0_0_54 — Big Chippy Gets a Mouth (2026-08-29)

### Added
- **Big Chippy (Info-card face) now has a mouth** that slowly drifts through emoji-style expressions — smile, flat, smirk (either side), open :D, small :o. It picks a new mood every ~5–9 s (much less often than the eyes) and gently morphs into it, so it reads as slow mood shifts rather than twitching. **Never frowns** — the curve only ever goes up or flat. Eyes keep their existing drift + downbeat X.

---

## V0_0_54 — Grid-Aligned Notes (2026-08-29)

### Fixed
- **Note blocks now land cleanly on the grid.** Previously, pitched voices (bass/lead/synth/accents) offset each block vertically by its pitch, so blocks floated at different heights within a lane and looked loose/misaligned. Every block is now a uniform height, centered on one consistent line per lane, pixel-rounded — reads like a proper step grid.

---

## V0_0_54 — About: Apple Music / AMU (2026-08-29)

### Changed
- Added **Apple Music** at the top of the Inspiration list, with a nod to the **Apple Music Understanding (AMU)** framework.

---

## V0_0_54 — About: Inspiration (2026-08-29)

### Changed
- **Removed the ROADMAP block** from the About tab (roadmap lives in git, not needed in-app).
- **Added an INSPIRATION section** in its place — currently Fatboy Slim and Smile (Ableton Live device).

---

## V0_0_54 — Party Evolves (2026-08-29)

### Changed
- **Party now evolves as it rocks.** It still starts seamlessly from whatever genre/bars are playing, but then keeps things interesting: every loop re-rolls a fresh pattern, ~1 in 3 loops hops to a new genre (and its tempo), ~1 in 4 loops varies the bar length. Mostly fresh patterns, occasional bigger changes — drifts and stays lively without flailing every bar.
- The genre and length dropdowns update to reflect where the party has wandered, so the UI stays honest.

---

## V0_0_54 — Party Rides Current Settings (2026-08-29)

### Changed
- **Party now keeps the current vibe going instead of overriding it.** It picks up whatever genre and bar length are already selected/playing and continues from there — no jarring switch to a random genre or a forced 64-bar song. Hit party while a House 16-bar loop is going and it stays House 16-bar, just re-rolling fresh pattern variations at each loop boundary until the timer's up.
- Controls are now **related, not independent**: the genre and length dropdowns drive both manual play and party. The length dropdown stays live during a party.
- The section drop-arrangement (`arrange`) no longer applies during party, so the full pattern keeps playing rather than thinning out.

### Removed
- The old party machinery: random genre switching (`GENRE_KEYS`), forced 64-bar songs (`PARTY_BARS`), per-song arrangement arc, and song counter — all replaced by "keep the current thing going with fresh variations."

---

## V0_0_54 — Roll Framing & All Voices On (2026-08-29)

### Changed
- **All voices default ON** — Accents and Fills were starting muted, which left their lanes looking empty/incomplete on load. They're part of the song, so they're on now.
- **Removed the dead black padding** at the bottom of the song roll. The roll height is now derived exactly from the lane count (7 lanes × 26px + small inner padding), so lanes fill the frame with no trailing gap.
- **Clearer, thicker frames** on both the **Chippy roll** (overview strip) and the **song roll** — each drawn as its own defined module (1.5px, brighter border) instead of relying on a faint outer container. The outer wrap border was removed so the two roll frames are the definition. Reads as two clean modular pieces.

### Naming
- Referring to the two lanes as the **Chippy roll** (the Chippy/overview strip up top) and the **song roll** (the main scrolling piano roll).

---

## V0_0_54 — Legend Removed (2026-08-29)

### Changed
- **Removed the color legend row beneath the piano roll** (DRUMS/PERC/BASS/…). It duplicated the Voices section, where each voice already shows its colored dot. Removed the element, its builder, and its CSS.

---

## V0_0_54 — Bottom & Helper Cleanup (2026-08-29)

### Changed
- **Removed the leftover footer CSS** (the status bar DOM was already gone; its styles are now gone too).
- **Removed the "7 lanes · scrolling playhead · pulse / triangle / noise" helper line** under the Piano Roll header — self-evident, not needed.
- **Moved the bar/beat counter into the Piano Roll section header** (right-aligned on the header rule), reclaiming the whole helper line.

---

## V0_0_54 — Footer Removed (2026-08-29)

### Changed
- **Removed the bottom status/footer bar** — it was redundant: the color legend sits directly above it and the play state is already clear from the transport and counter. `setMsg()` is retained (guarded) so nothing breaks; it just no longer paints a footer. Cleaner bottom edge.

---

## V0_0_54 — Keyboard-Focusable Wheels (2026-08-29)

### Changed
- **All wheel pickers (tempo, swing, voice levels) are now keyboard-focusable.** Tab moves focus from one to the next; a cyan focus ring shows which is active. Adjust with **Up/Right (+) and Down/Left (−)**, hold **Shift for ×10** steps. Scroll and drag still work. You can now set everything without leaving the keyboard.

---

## V0_0_54 — Voice Level Wheels (2026-08-29)

### Changed
- **Voice volume sliders replaced with wheel pickers** — the same scroll/drag control used for tempo and swing. Hover a voice's level and scroll, or drag up/down. No more fiddly slider thumb or awkward keyboard/arrow behavior. 0–100, clamped.

---

## V0_0_54 — Slimmer Voice Cards (2026-08-29)

### Changed
- **Voice cards are now a single tight row** — dropped the name-label row above each card to reclaim vertical space. Each card is just: colored LED (click to mute) · oscillator select · volume · value.
- The voice **name and what it does now live in a hover tooltip** on the card; the colored dot still identifies the voice (matching the legend below).

---

## V0_0_54 — Counter Moved (2026-08-29)

### Changed
- Moved the **bar/beat counter** out of the middle of the Controls row (where it was forcing the row to wrap) into the **Piano Roll** header line, right-aligned. Controls row stays on one line; cleaner.

---

# OMS Chippy [Beta V0_0_54] — COMPLETE CHANGELOG

**License:** GPL-3.0

---

## V0_0_54 — Party Song-Clock Fix (2026-08-29)

### Fixed
- **Party mode was spawning a new song every frame** — the "next song" trigger watched the playhead position wrap (`p < prevPos`), but right after each song reset (with `loopStartCtx` set slightly in the future and the scheduler mutating it), that comparison fired instantly and repeatedly, so party got stuck thrashing a tiny fragment.
- Party now advances on an **explicit audio-time song clock**: each song anchors `songStartCtx` and stores its full 64-bar `songDur`; `partyTick` rolls the next song only when `actx.currentTime - songStartCtx >= songDur`. Immune to the scheduler's loop bookkeeping.

### Note
- At real dance tempos a 64-bar song runs ~1.5–2 min (124 bpm ≈ 2:04, 174 bpm ≈ 1:28), longer than the earlier 30–60 s estimate. If shorter songs are wanted, dropping to 32 bars would roughly halve it.

---

## V0_0_54 — Party Mode 🎉 (2026-08-29)

**The 1.0 milestone: Chippy becomes a toy you leave running.**

### Added
- **PARTY button** — a subtly squishy, pulsing button (jelly-scale breathing, gentle glow) that's meant to be irresistible to press. Click it and Chippy spins an endless set.
- **Duration picker** to its right — 5 / 10 / 20 / 30 / 60 min. Sets total party length.
- **Party engine** — generates back-to-back **fixed 64-bar songs**, each a random genre (House / Breaks / Drum & Bass / Big Room) with a fresh seed. Each song plays start to finish (~30–60 s depending on tempo), then the next spawns at the clean boundary. Runs until the timer's up, then stops gracefully and restores manual mode.
- **Section-based arrangement** for 64-bar songs (`arrange`) — an 8-section arc gives each song a real shape: sparse intro (drums+bass) → build → full DROP → no-kick breakdown → build → DROP → outro. Songs breathe like tracks instead of looping 4-bar patterns.
- Live party readout: Info shows "Party · <genre> · Song N"; the time display shows the countdown and current bar (e.g. "party 8:32 · bar 17/64").

### Notes
- During party, the length dropdown is inert (songs are fixed 64 bars); it's restored on stop.
- Song changes land on the downbeat by construction (each song is its own boundary), so transitions are always musical — no crossfade needed.
- Chippy's reactive faces run throughout the party.

---

## V0_3_4 — Chippy's Face (2026-08-29)

### Added
- **Chippy's face in the Info card** (right side, echoing the original Sozo layout). He sits still but reacts to the music every frame:
  - **Kick** → mouth opens and the whole face squishes/pulses on the beat
  - **Bar downbeat** → eyes flip to X (matching the conductor up top), dots otherwise
  - **Snare/clap** → eyebrow raise + a quick wink
  - **Bass** → eyes puff a touch, face widens
  - **Lead** → eyes glance side to side
  - **Crash** → big open "whoa" mouth + sparkle burst around his head
  - At rest → calm grin
- Face reactions are driven by a per-type hit-tracker (`detectHits`) that stamps timestamps as the playhead crosses events, with a decay envelope (`env`) animating each feature by recency.

### Also
- Removed the "length" label from the control strip (dropdown is self-evident, matching the earlier "style" removal).

---

## V0_3_3 — Wheel Pickers & Panel Sections (2026-08-29)

### Changed
- **Tempo and swing are now compact wheel pickers** instead of sliders — scroll over them or drag vertically to change. Saves horizontal space in the control strip. Same ranges (tempo 70–180, swing 0–60%).

---

## V0_3_2 — Back to WAV (2026-08-29)

### Changed
- **Export reverted to WAV** (native 16-bit mono PCM). Removed the inlined lamejs MP3 encoder and its About acknowledgment. File is back to ~49 KB, single-file with **no bundled dependencies**, fully offline. WAV is lossless and imports straight into any DAW.

### Notes
- V0_3_1 (MP3) is retained in history as the detour. Chippy's no-dependencies principle is intact again.

---

## V0_3_1 — MP3 Export (2026-08-29)

### Changed
- **Export is now MP3** (192 kbps, mono). The offline render is unchanged; the rendered buffer is encoded to MP3 via **lamejs**, inlined into the HTML.
- Button relabeled "export mp3". Filename e.g. `Chippy_House_16bar_124bpm.mp3`.

### Added
- Inlined `lamejs` 1.2.1 (LGPL-3.0, built on LAME) — ~156 KB, unmodified. This is the first bundled third-party code in Chippy; it keeps the app single-file and fully offline. Acknowledged in the About tab with a link to LAME per LGPL.

### Notes
- Conscious tradeoff: browsers ship no native MP3 encoder, so MP3 requires a bundled encoder. The file grows to ~205 KB but remains standalone and dependency-free at runtime (nothing fetched). WAV (V0_3_0) was native/lossless; MP3 is the requested format.

---

## V0_3_0 — WAV Export (2026-08-29)

### Added
- **Export WAV** button in the control strip. Renders the current loop offline via `OfflineAudioContext` (faster than real-time) using the exact same synth voices, then encodes 16-bit mono PCM WAV by hand and downloads it. Filename encodes style, length, and BPM (e.g. `Chippy_House_16bar_124bpm.wav`).
- Native, single-file, no dependencies, offline — no third-party encoder bundled.

### Changed
- Removed the "style" label from the control strip (the dropdown is self-evident).

### Notes
- WAV, not MP3: browsers ship no native MP3 encoder, and bundling one (e.g. lamejs) would add ~100 KB of third-party code against the no-dependencies rule. WAV renders instantly, is lossless, and drops straight into Ableton or any DAW. An inlined-MP3 option can be added later if specifically needed.

---

## V0_2_4 — Layout Polish (2026-08-29)

### Changed
- **Control strip regrouped** — Length now sits right after Style (the two "what to make" dropdowns together), then Generate · Play · time · tempo · swing, with thin dividers separating the groups.
- **Tightened alignment** — consistent spacing and baselines across the control strip; dropdowns share one style; the hint moved to the right edge.
- **Piano-roll framing** — the overview strip and roll are now one cleanly-bordered unit. Chippy's strip lost its odd standalone outline; a soft divider ties it to the roll below, and the whole roll sits in a proper frame instead of floating.

*(A future "loose" toggle to restore the scattered look is parked — not built.)*

---

## V0_2_3 — Unified Center Alignment (2026-08-29)

### Changed
- **One scroll behavior at every length.** The playhead is always dead-center and the roll is always positioned around it — including 4-bar loops. Removed the "fits the window" special case, so there's no branch: the current step simply sits at window center, always.
- The roll now **opens aligned** — at rest the playhead is already centered on bar 1 (playhead and Chippy shown at rest), so the view makes sense before you even press play.

---

## V0_2_2 — Center-Locked Scroll (2026-08-29)

### Changed
- The piano roll now **scrolls from the start** whenever the loop is longer than the visible window (4 bars), with the **playhead fixed dead-center** and the roll sliding under it — no more waiting to reach an edge, no end-clamp. Loops that fit the window (4 bars) still don't scroll.

### Removed
- The cyan viewport outline box in the overview strip — Chippy alone marks the position now.

---

## V0_2_1 — Chippy the Conductor (2026-08-29)

### Added / Changed
- The overview smiley is now **Chippy the conductor** — yellow face with black features, riding the loop left to right.
- **Eyes flip dots → X on the bar downbeat** and return to dots between bars. (Default face is now dot eyes; X-eyes are the on-the-bar beat tell.)

---

## V0_2_0 — Length + Scrolling Roll + Smiley Overview (2026-08-29)

### Added
- **Length control** — a dropdown in the control strip: 4 / 8 / 16 / 32 bars. Changing length regenerates the pattern at that length in the current genre.
- **Scrolling piano roll** — bars now hold a fixed readable width regardless of loop length; the roll scrolls horizontally to follow the playhead (visible window = 4 bars), so elements never shrink for longer loops.
- **Overview strip** above the roll — a full-length bar showing the whole loop, with bar ticks, a cyan viewport box marking the currently-visible window, and an **X-eyes smiley** that rides along the strip to show loop position.

### Changed
- Canvas layout split into an overview strip + a scrolling main roll. `BARS` / `TOTAL_STEPS` are now variable; step→x uses a fixed pixels-per-step plus a scroll offset that anchors the playhead ~1/3 from the left.

---

## V0_1_1 — Genre Engine + Draw Fixes (2026-08-29)

### Added
- **Genre engine** — four dance styles selectable from a control-strip dropdown, each with its own kick / backbeat / hat / bass rules and default tempo:
  - Big Room (128) — 4-on-floor, big triad stabs, sparse anthem lead
  - House (124) — 4-on-floor, offbeat bass ducking the kick, ghost-hat swing
  - Breaks (130) — syncopated breakbeat (not 4-on-floor), snare on 2 & 4, busy hats
  - Drum & Bass (174) — two-step kick + snare, fast broken hats, long sub rolls
- **Downbeat anchoring** across all genres — accented kick on beat 1, phrase-top crash, clap/snare on 2 & 4, last-bar fill into the next 1.
- **Dedicated kick synth** (`playKick`) — sine body with a fast downward pitch sweep + highpassed noise click; the Drums lane routes through it.
- **Accent / snare / crash flags** carried through the event list into playback (louder anchored kick, bright clap/snare bursts, phrase-top crash wash).
- Header greeting text set to "Hi Friend ;)".

### Fixed
- **Lead density** — dense euclidean lead that rendered as a solid wall is now sparse per genre; reads as a melody.
- **Synth chord clipping** — tall chord blocks that spilled below their lane into the Accents row are now contained; note vertical positioning rewritten with a per-lane band and hard clamp.
- **Lane labels** — were overdrawn by notes; now drawn last on a dark backing chip, always legible.
- **Breaks / DnB downbeat** — the accented "1" is now a genuine boss kick (louder ×1.35, deeper 190→40 Hz sweep, longer tail) on *every* bar, and fast hats are cleared off the downbeat so the kick transient isn't masked. The 1 punches through the syncopation.

### Changed
- **Default style: House** (was Big Room). Style pick sets tempo and re-rolls in-style; Generate / G re-rolls within the selected style.

---

## V0_1_0 — Engine & Roll (2026-01-22)

### Added
- First build. Chiptune engine on the Sozo visual chassis, all Music Understanding / AMU mechanics removed.
- **7-lane engine** — Drums, Perc, Bass, Lead, Synth, Accents, Fills; each an independent voice.
- **Synthesis (Web Audio)** — variable-duty pulse via `PeriodicWave` (12.5 / 25 / 50%), triangle, and a real noise channel (bandpassed white-noise buffer source), each with a snappy envelope.
- **Lookahead scheduler** off `AudioContext.currentTime` (0.12 s horizon, loop-wrapping) — drift-free.
- **Piano roll** on canvas — 7 lanes, bar/beat/16th grid, pitch-positioned notes, scrolling playhead.
- **Transport** — play/pause, tempo (70–180), swing (0–60%). **Generate** re-roll with random transpose. **Voice strip** — per-voice toggle, oscillator select, level. Keyboard: Space = play/pause, G = generate.

### Fixed (in build)
- Root tuning corrected (A minor now sits at A3 = 220 Hz; was landing at F♯ due to a double-counted tonic offset).
