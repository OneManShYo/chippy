# OMS Chippy [Beta V1_1_0] — USER GUIDE

**Version:** Beta V1_1_0
**Date:** August 29, 2026
**License:** GPL-3.0 (software)

---

## 1. WHAT CHIPPY IS

Chippy is a chiptune EDM generator that runs in the browser. It builds dance loops out of classic chip voices — pulse, triangle, and noise — and plays them back on a scrolling piano roll. Pick a genre, press play, and re-roll for a fresh loop whenever you want. No DAW, no files, no install.

## 2. THE TABS

- **GENERATE** — the main view: loop info, control strip, the piano roll, the voice strip.
- **ABOUT** — background on the tool and the ONEMANSHYO project.

## 3. THE CONTROL STRIP

Left to right:
- **Style** — the genre: Big Room, House, Breaks, or Drum & Bass. Picking one sets its tempo and generates a fresh loop in that style.
- **⚄ generate** — re-rolls the pattern in the current style. (Keyboard: **G**.)
- **▶ play / ❚❚ pause** — start/stop. (Keyboard: **Space**.)
- **tempo** — 70–180 BPM, a wheel picker: scroll over it or drag up/down. Changing style resets tempo to that style's default.
- **swing** — 0–60%, a wheel picker (scroll or drag); delays the offbeat 16ths for a looser feel.
- **length** — loop length: 4 / 8 / 16 / 32 bars. Changing it regenerates the pattern at that length in the current style.

## 4. THE GENRES

- **Big Room** (128) — four-on-the-floor kick, big sustained chord stabs, sparse anthem lead.
- **House** (124) — four-on-the-floor with an offbeat bassline that ducks around the kick, plus ghost-hat swing.
- **Breaks** (130) — a syncopated breakbeat (not four-on-the-floor) with the snare on 2 and 4.
- **Drum & Bass** (174) — a fast two-step kick-and-snare with broken hats and long sub-bass rolls.

Every style keeps beat 1 obvious — the kick is accented on the 1, a crash marks the top of the phrase, and the last bar fills into the next loop.

## 5. THE OVERVIEW STRIP

Above the roll, a bar spans the full loop length with bar-number ticks. Chippy — the yellow conductor — rides along it showing where you are in the loop, his eyes flipping from dots to X on each bar's downbeat, and a highlighted box marks the part the scrolling roll is currently showing. On a 4-bar loop the whole thing fits and the roll doesn't scroll; on longer loops the strip is your big-picture map.

## 6. THE PIANO ROLL

Seven lanes, top to bottom: **Drums, Perc, Bass, Lead, Synth, Accents, Fills.** Notes draw as colored blocks; pitched lanes place notes higher or lower by pitch. Bars stay a fixed readable width — on loops longer than 4 bars the roll scrolls to follow the magenta playhead, so notes never shrink. Accents and Fills ship off by default — turn them on in the voice strip.

## 7. THE VOICES STRIP

Below the roll. For each lane:
- **LED / name** — click to toggle the voice on/off.
- **Oscillator** — Pulse 12% / 25% / 50%, Triangle, or Noise.
- **Level** — the voice's volume.

## 8. KEYBOARD

- **Space** — play / pause
- **G** — generate a new roll

## 9. REQUIREMENTS

Any modern browser (Chrome recommended), any OS. No install, no dependencies. Sound starts on your first click (browsers block autoplay until then).
