# OMS Chippy [Beta V1_15_0] — USER GUIDE

Chippy is a browser instrument. Open the file, press play, and it makes music. No install, no account,
no files. Here's everything you can do.

## The three tabs
- **MUSIC** — the instrument (make and shape loops). (Renamed from TRACKS.)
- **GLOVER** — the full-screen light show. Pick a trick, watch the Chippy family rave.
- **ABOUT** — what it is, who made it, PLUR.

## The message bar + monitor
- **Message bar** (bottom of the app) — hover any control and it tells you what it does; change a value
  and it shows the new value. Cyan = help, orange = a value you changed, magenta = an error.
- **Monitor** (top row, center) — the live readout: **master** clock, **music** (loop position / loop total,
  e.g. `9 . 1 . 1 / 64 . 4 . 4`, blinking through its last bar), and **tempo**. When a Lineup is running the
  music row appends a countdown to the next selecta (`⇢ b . b . 16`). A fourth **playing** line scrolls what's
  on now — and, mid-Lineup, what's up next. Hover any row (or Chippy's face) for a one-line explainer.
- **MATRIX orientation** — the button by the MATRIX label flips the piano roll between vertical (default:
  time flows up, voices as columns under their mixer channels) and horizontal (time flows left). The
  overview strip with Chippy stays put and shows your position in the whole loop either way.
- **Selecta DNA** — picking a selecta conditions the generator with a musical profile; None plays raw. Two
  ship: **Resident DJ** (the default — the always-available house act, a corpus-free hand-authored profile)
  and **Juice Night Out** (built from analysis of Wes's own tracks). While a selecta runs, the fields it
  controls (genre/bars/tempo/swing) disable, since it's driving them.

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

## The Lineup (session set-list)
Click the **notepad button** in the SELECTA row to open the **Lineup** — a slide-out where you program a
set. Add slots; each slot picks a selecta (Resident DJ, Juice Night Out) and a duration in minutes (scroll
or type). Hit **Play Lineup** and Chippy runs the slots in order, swapping DJ on the bar line when each
slot's time is up. A pink ring glows on the button while a lineup plays, the monitor shows who's up next,
and the music row counts down to the swap. The lineup lives for the session only — it resets on reload.

## Voices (the 8 modules)
Each voice (drums, perc, bass, lead, synth, accents, fills, FX) is a module with a **sound selector** and a
**dice** button (re-roll that voice's sound). The **FX** voice makes risers, glitches, impacts, sweeps, and a
ship horn — it's a *sound*, one of the eight instruments. (Not to be confused with the mixer's audio
effects, which are processing — see below.)

## The mixer (levels + live DJ effects)
Below the voices is the mixer. Each of the 8 channels has a **level**, a **solo** (S), and a **kill** (⏻).
Solo is additive — tap S on a few channels to build a group; tap again to drop one. The **master solo** on
the mixer header lights whenever anything is soloed; one tap clears them all (great for a subtractive
build — solo a few elements down, then release everything on the drop).

The mixer header carries the **live DJ effects**, each in its own little module (label + button + amount):

| Control | What it does | How to use |
|---|---|---|
| **ECHO** | Tempo-synced repeat (dotted-eighth), bass cut so it stays clean. | Button on, dial the amount. Great for transitions/exits. |
| **FILTER** | The DJ sweep. Dial down = muffled/underwater; up = thin/bass-out; 5 = off. | Turn to sweep; button toggles it in/out and *keeps* your setting. |
| **BACKSPIN** | Tape-stop brake — the mix spins/pitches down while held. | **Press and hold** (or hold **Enter** when focused); release and you're back on the beat. Dial = brake length. |

At the far right of the mixer header is the **Master panel button** (the cyan-glow panel icon) — it opens the
Master slide-out.

## The Master panel (level, output, DSP, effects)
Press **M** or the Master button on the mixer header to slide out the **Master panel**. It's a non-blocking
overlay — the app keeps running behind it, so you can open it and keep playing. Close it with the **✕**, the
button again, or **Esc**. Top to bottom:

- **LEVEL** — a stereo L/R meter with a dB scale (peak-hold marks + clip dots), and **OUTPUT** / **PEAK**
  readouts. This is your "how hot is the mix" at a glance.
- **OUTPUT** — **Talk**, **Volume**, **Mute**. **Talk** dims the master ~10 dB so you can talk over the
  music (tap again to restore). **Volume** is the master level. **Mute** silences the mix without stopping
  playback.
- **DSP** — **EQ** (High/Mid/Low, ±12 dB, 0 = flat), **Comp** (Threshold / Ratio / Attack — a real
  compressor to glue and thicken), **Limiter** (adjustable Ceiling — the peak ceiling so the output can't
  clip). Hover any control and the message bar tells you what it is.
- **Audio Effects** — **Reverb** (space/wash on the mix; drums stay dry). Button on/off, dial for strength.


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
