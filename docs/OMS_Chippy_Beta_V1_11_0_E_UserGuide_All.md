# OMS Chippy [Beta V1_11_0] — USER GUIDE

Chippy is a browser instrument. Open the file, press play, and it makes music. No install, no account,
no files. Here's everything you can do.

## The three tabs
- **MUSIC** — the instrument (make and shape loops). (Renamed from TRACKS.)
- **GLOVER** — the full-screen light show. Pick a trick, watch the Chippy family rave.
- **ABOUT** — what it is, who made it, PLUR.

## The message bar + monitor
- **Message bar** (bottom of the app) — hover any control and it tells you what it does; change a value
  and it shows the new value. Cyan = help, orange = a value you changed, magenta = an error.
- **Monitor** (top row, center) — the live readout: master clock, loop clock, tempo. The loop clock
  blinks through its last bar so you can see a change coming.

## Making music (MUSIC)
- **Play (▶)** — start/stop.
- **Randomize (⚄)** — generate a fresh loop. Each press = a new tune with a new name.
- **Kill (⏻)** — mute the whole mix without stopping (talk over it, then unmute).
- **Length / Tempo / Swing** — shape the current loop. Length is a "new song" change (it lands cleanly
  at the loop end); tempo & swing change live like knobs. (Genre is House-only this release.)

## Party mode (the auto-DJ)
Press **party** and Chippy runs a nonstop set on its own, evolving loops seamlessly. Choose:
- **Selecta (style)** — the DJ / record crate. Currently one ships: **Juice Night Out**, a house
  selecta modeled on the real Juice Night Out label sound (built from analysis of Wes's own tracks).
  In party mode it conditions the set toward that sound — drum-forward arrangement, DJ moves at the
  phrase turns, and tempo that creeps rather than jumps.
- **Time slot** — the club-night arc: Opener → Support → Headliner → Closer (quieter/patient →
  loud/peak → wind-down).

Party is where the YoConditioner runs — the layer that biases what gets generated toward the selecta's
sound. Outside party mode, generate gives you clean, un-conditioned house.

## Voices (the 8 modules)
Each voice (drums, perc, bass, lead, synth, accents, fills, FX) is a module with a **sound selector** and a
**dice** button (re-roll that voice's sound). The **FX** voice makes risers, glitches, impacts, sweeps, and a
ship horn — it's a *sound*, one of the eight instruments. (Not to be confused with the mixer's audio
effects, which are processing — see below.)

## The mixer (levels, effects, processing)
Below the voices is the mixer. Each of the 8 channels has a **level**, a **solo** (S), and a **kill** (⏻).
Solo is additive — tap S on a few channels to build a group; tap again to drop one. The **master solo**
button lights whenever anything is soloed; one tap clears them all (great for a subtractive build — solo a
few elements down, then release everything on the drop).

Along the mixer header is the **audio-effects rack** and the **master processing**, each control labeled so
you know what it is at a glance. Every dial runs a simple **0–10** scale (EQ is in dB).

**Audio effects (live performance FX on the whole mix):**

| Control | What it does | How to use |
|---|---|---|
| **REVERB** | Space/wash on the mix (drums stay dry). | Button on/off, dial for strength. On by default, low. |
| **ECHO** | Tempo-synced repeat (dotted-eighth), bass cut so it stays clean. | Button on, dial the amount. Great for transitions/exits. |
| **FILTER** | The DJ sweep. Dial down = muffled/underwater; dial up = thin/bass-out; 5 = off. | Turn the dial to sweep; button toggles it in/out and *keeps* your setting. |
| **BACKSPIN** | Tape-stop brake — the mix spins/pitches down while held. | **Press and hold** (or hold **Enter** when focused); release and you're back on the beat, no time lost. Dial = how long the brake takes. |

**Master processing (set-and-forget session tone, end of the chain):**

| Control | What it does | How to use |
|---|---|---|
| **EQ** (High/Mid/Low) | 3-band tone shaping on the whole mix. | Each dial ±12 dB, 0 = flat. Boost/cut top, middle, or bottom. |
| **COMP** | Compressor — glues/thickens the mix. | Button on/off; dial sets the ratio (gentle → hard). Gets louder/fatter as you push it. |
| **LIMIT** | Limiter — a peak ceiling so the output can't clip. | Button on/off. Safety net; catches whatever the effects throw at it. |
| **VOL** | Master output volume (live). | The final level, at the very end before kill. |

**Kill (⏻)** — mute the whole mix without stopping playback (talk over it, then unmute).

## Glover (the light show)
Go to GLOVER, hit party, and pick a trick: Tracers, Liquid, Tutting, Strobe, Orbits, Bloom. Each is a
different visual. You can record it (the record button, top-right) to your camera roll / downloads,
and toggle 16:9 or 9:16 orientation.

## Keyboard (power users)
- **Space** = play, **G** = generate, **N** = generate.
- **WASD** = move around the controls/voices grid; **arrow keys** = change the focused control.
- **Enter** = trigger the focused button. Hold **Enter** on **BACKSPIN** to keep the brake engaged (release to let go).
- **Tab** works normally too.

## Tips
- Turn on **persistence (∞)** to keep the music playing when you switch tabs.
- Chippy runs great as ambient "fill the room" music — hit party and let it go.
