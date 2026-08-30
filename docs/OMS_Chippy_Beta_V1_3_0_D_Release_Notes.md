# OMS Chippy — Release Notes

## V1_3_0 (2026-08-29)

**Performance & Recording release.** This is a minor-version roll that closes out the long V1.0 iteration line as one substantial feature release. Highlights:

- **Play it like an instrument with the keyboard** — WASD moves around the Controls and Voices grid, arrow keys change the focused control live. Tab still works as normal.
- **A running session counter** in Ableton's bars.beats.sixteenths style sits next to the loop position, so you always know how far into a take you are.
- **Recording is fixed for long takes** — minute-plus captures now save as clean, playable files (real MP4 where the browser supports it), and the record controls stay put on every tab.
- **Smoother live tweaking** — tempo and swing now change on the fly without restarting the loop, while length waits for the current loop to finish before switching.

Live at chippy.onemanshyo.com.

---

## V1_0_14 (2026-08-29)

**Fix — beat-quantize clean cut.** Changing a control while Chippy is playing now applies exactly on the next beat with no audible skip. Earlier the swap could be computed past the scheduler's look-ahead window and fail to apply; it's now anchored to a fixed playback origin and an absolute audio-clock target, so it always lands on the beat and the loop restarts cleanly.

Live at chippy.onemanshyo.com.

---

## V1_0_1 (2026-08-29)

First iteration on the V1.0 release series.

**Fix:** The ∞ button is now correctly labelled a **persistence** control — it keeps playback alive when you switch to another browser tab, rather than being described as "continuous play." Tooltip and status messages corrected. No functional change to the mechanism.

Live at chippy.onemanshyo.com.
