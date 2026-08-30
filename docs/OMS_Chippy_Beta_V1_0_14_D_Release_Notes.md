# OMS Chippy — Release Notes

## V1_0_14 (2026-08-29)

**Fix — beat-quantize clean cut.** Changing a control while Chippy is playing now applies exactly on the next beat with no audible skip. Earlier the swap could be computed past the scheduler's look-ahead window and fail to apply; it's now anchored to a fixed playback origin and an absolute audio-clock target, so it always lands on the beat and the loop restarts cleanly.

Live at chippy.onemanshyo.com.

---

## V1_0_1 (2026-08-29)

First iteration on the V1.0 release series.

**Fix:** The ∞ button is now correctly labelled a **persistence** control — it keeps playback alive when you switch to another browser tab, rather than being described as "continuous play." Tooltip and status messages corrected. No functional change to the mechanism.

Live at chippy.onemanshyo.com.
