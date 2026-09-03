# OMS Chippy [Beta V1_13_0] — RELEASE NOTES

**Live at chippy.onemanshyo.com**

## V1.13.0 — Resident DJ, the Lineup, and a smarter monitor

This release makes Chippy programmable as a set. The headline is the **Lineup** — a styled slide-out where
you build a session set-list: pick which selecta plays each slot and for how many minutes, hit Play Lineup,
and Chippy runs the whole thing on its own, swapping DJs on the bar line as each slot's time runs out. It's
a scheduler sitting on top of the music engine — it decides who's on when, and never touches how the music
is generated.

Alongside it arrives **Resident DJ** — the house act. It's the original pre-JNO "Giorgio Levan" selecta
brought back as a permanent, always-available conditioned set, built from a tiny hand-authored profile with
no analysis behind it at all. It proves the point that a selecta doesn't need a corpus of stems and MIDI to
work — a few honest values are enough. Resident DJ is now the default when Chippy opens.

The **monitor** got a real upgrade to match. A new "playing" line scrolls what's on now and, when a lineup
is running, what's coming up next — like a booth monitor or a jukebox up-next. The counter row (now labeled
"music") shows a live countdown to the next selecta in Chippy's own bar-beat-sixteenth format. And a pulsing
pink ring glows on the Lineup button whenever a set is running, so you always know it's live. Every monitor
row and even Chippy's face now explain themselves on hover.

Under the hood, the controls that a selecta takes over — genre, bars, tempo, swing — now disable themselves
while a selecta is driving (with keyboard nav skipping right past them), and a handful of interface papercuts
got cleaned up along the way.

## V1.12.0 — Yo DNA, the section engine, and a vertical roll

This release is the first real payoff of the YoConditioner: Chippy now generates with **Wes's own musical
DNA** baked in. Selecting the Juice Night Out selecta pulls in the actual programmed patterns from a real
track — the four-on-the-floor kick, the dead-straight offbeat hats, the E-minor sub-bass rumble, the
backbeat clap — folded out of the original session and used to bias the generator. It doesn't replay the
song; it leans the endless generation toward that feel. You can hear the difference the moment you switch
from None to JNO.

Underneath that sits the **section engine**: Chippy can now shape a whole arrangement — intro, build,
verse, breakdown, drop, outro — instead of looping one flat idea. Because Chippy plays continuously and
never mixes, the long intros and outros that only exist to give a DJ headroom get trimmed automatically,
and the meat of the track breathes into whatever loop length you've set. It's a bias on top of the live
generator, never a fixed script — two passes of the same shape still differ in the detail.

On the interface side, the piano roll can now run **vertical** — time flowing bottom to top with the eight
voices as columns lined up under their mixer channels, like a channel grid. A toggle in the MATRIX header
flips between that and the classic horizontal view. The loop counter also now shows how long the current
loop is (e.g. `9 . 1 . 1 / 64 . 4 . 4`), so you always know where you are in the cycle.

This release also fixes two timing issues around switching into a conditioned set, so the roll stays locked
to the audio through the change.

## V1.11.0 — The performance mixer

This release turns the mixer into a real performance instrument. The headline is a **master audio-effects
rack** — four live effects sitting across the mixer header, each a labelled button and a simple 0–10 dial:

- **Reverb** — space on the whole mix (drums stay dry). On by default, light — just enough to know the
  effects are there.
- **Echo** — a tempo-synced repeat that trails a phrase off for transitions, with the bass rolled out of
  the repeats so it stays clean.
- **Filter** — the classic DJ sweep. Turn it down for that muffled, underwater drop; turn it up to pull the
  bass out and build tension. The button punches it in and out while keeping your setting.
- **Backspin** — a tape-stop brake. Press and hold (mouse, finger, or the Enter key) and the whole mix
  spins and pitches down like slowing a record; let go and you're right back on the beat, no time lost.

After the effects sits a slim **master processing** chain for setting your tone once and leaving it: a
three-band **EQ** (high/mid/low), a **compressor** to glue and thicken, a **limiter** as a peak ceiling so
nothing clips, and the **master volume** at the very end. It's built from the same DSP recipe as OMS Dojo,
trimmed down for performing rather than mastering — everything starts flat and transparent until you dial
it in.

There's also a **master solo** move: solo a few elements down over a phrase, then hit one button to release
them all and slam the full mix back in on the drop.

Under the hood, the whole app got more keyboard-first — every control now lights the message bar when you
tab or arrow onto it, not just on mouse hover, which also sets up external controllers down the road. The
internal naming was cleaned up so \"FX\" always means the FX *sound* (in the matrix) and never the audio
*effects* (the processing) — no more crossed wires. And a headless boot-check now guards every change, so
the effects rack shipped without a single broken build.

A couple of housekeeping notes: the About panel now spells out that the icons, music, and MIDI are
proprietary (the GPL-3.0 covers the source code only), and the auto-DJ (YoConditioning) no longer applies
audio effects automatically — that waits until the new effects are better understood. It still handles the
arrangement moves that make a set breathe.

## V1.10.0 — The Message System, the Monitor, and a proper UI pass

This release is about the interface growing up. The headline is a real **message system**, ported from
OMS Dojo: a feedback bar across the bottom of the app. Hover any control and it tells you what that
control does; change a value and it shows you the new value. It replaces scattered browser tooltips with
one consistent, in-app channel — and it's managed centrally, so every label and message lives in one
place. Cyan for help, orange for a value you just changed, magenta for an error.

The center of the top row is now the **Monitor** — a live readout of what the music is doing right now:
the master clock, the loop clock, and the tempo, laid out to match the Info panel beside it. Info tells
you what the track *is*; the Monitor tells you what it's *doing*. The loop clock blinks through its last
bar so you can see a change coming.

The controls got a real structure pass. The old "Controls" row is now two clearly-labelled sections
sharing a row — **Transport** (play + generate) and **Selecta** (name, genre, time slot, bars, tempo,
swing, quantize) — so it reads like gear, not a pile of dropdowns. The **Mixer** gained a per-channel
**Solo** (replacing the per-channel reverb, since reverb belongs on the master bus), the master effects
are now clearly labelled **Audio Effects** (distinct from the FX *voice* in the piano roll), and the
master **Mute** is labelled. The **Voices** header now shows the **Source** — the hardware Chippy's
sound is modeled after (a Ricoh 2A03, the NES/Famicom chip).

The first tab was renamed from **Tracks** to **Music**. And a design-system pass brought field colors
back in line: value text is neutral, with cyan reserved for headers, focus, and the live readouts where
it actually means something.

Under the hood, two long-standing visual bugs were fixed for good: the conductor "Chippy" on the overview
strip no longer vanishes early or jumps at the loop boundary (he now rides the master clock like
everything else), and the loop/session clocks were split into their own clean readouts.

## V1.9.0 — One play button, one continuous roll

This release finishes what removing "party mode" started. There's now a single **play** button and the
**selecta** dropdown is the mode switch: leave it on **None** and Chippy just plays — a raw loop, no
conditioning; pick **Juice Night Out** (or any selecta) and it runs a conditioned nonstop set in that
style. One fewer control, and no more guessing what "play" versus "party" did.

Switching a selecta now behaves like a real DJ handing off: it **lets the current bar finish**, then the
new selecta drops in on the bar line — no mid-bar cut. That's true whether you're starting from None or
moving between selectas.

The piano roll is now **continuous** once you press play. It starts cleanly at bar one with the loop laid
out ahead, and from then on it scrolls without ever blanking — the loop's own beginning comes back around
at the tail. Under the hood, the visual and the audio now run off one shared clock, which also fixed a
handful of rough edges: the play/pause button no longer resizes and nudges the row, stopping truly resets
to the top instead of freezing mid-loop, the playhead no longer races ahead of the sound on the first
play, first-play starts on time with no dead beat, and the roll no longer jumps at each loop boundary.

## V1.8.0 — The Mixer (and a documented Design System)

This release splits Chippy's console out from its instrument. The old "Voices" section did two jobs at
once — it held both the *sound* of each voice (which kick, which oscillator) and the *mixer* controls
(level, reverb, mute). Those are two different machines: the instrument you load, versus the hands you
run over a mixer. V1.8.0 separates them into two rows — **Voices** (selector + dice) and a new **Mixer**
(level + reverb + kill per channel) — lined up lane-for-lane. It's the same split a real DJ rig keeps:
the decks that make the sound, and the mixer you perform on. It also sets up the next step — the
YoConditioner's automatic DJ moves will visibly fire on the Mixer, since that's where mixing lives.

Alongside that, the modules were cleaned up (uniform heights, readable selectors, consistent padding)
and the whole layout tightened so it fits one screen. And the **Design System** — the colors, spacing,
type, and component rules that keep Chippy looking like OMS — is now written down as a core spec, so the
look stays consistent as the app grows.

Also fixed: iOS audio now routes to AirPlay / HomePod correctly, verified on device.

## V1.7.0 — The YoConditioner

This is the release where Chippy's DNA starts to actually shape the sound. Until now, a selecta's
profile only nudged groove and key; everything else it "knew" about Wes's music was captured but
silent. V1.8.0 turns that on.

**The YoConditioner** is the name for the layer that decides *what* gets generated — the generative
brain that sits above the sound set (chiptune today, swappable tomorrow). It runs on two ideas, kept
deliberately separate: **Yo DNA** is the committed fingerprint of a style (measured from Wes's own
tracks, or authored from his years of DJing); **Yo Conditioning** is the act of using that DNA to bias
what the generator produces. It isn't training and it isn't sampling — it tilts the odds so the music
leans toward the reference, while every loop is still generated fresh.

**Juice Night Out is the proof.** Its profile was rebuilt from real analysis of 12 of Wes's own house
tracks (now recorded in the profile itself as its corpus), and three things now flow from that DNA into
the sound during party mode:

- **It arranges drum-forward, like the source.** Each instrument's busyness is set by its measured
  activity — drums stay full, bass and melody sit back in the exact proportion the real tracks show.
- **It performs, not just plays.** The DJ moves are in: kick cuts, bass EQ-outs, lead drops, and reverb
  throws land at phrase turns, each on a probability so they feel human, not clockwork.
- **It builds energy by creeping the tempo.** Between songs the BPM only nudges a couple of beats at a
  time — the way you actually walk a room up, instead of jumping.

To keep the focus tight while this is dialed in, Chippy is **House-only** this release — the other
genres were pulled from the code and will come back rebuilt to the new standard. The YoConditioner runs
in **party mode**; plain generate still gives you clean, un-conditioned house.

## V1.6.0 — Juice Night Out

This release refocuses the selecta system around a single, real profile. The former Giorgio Levan
selecta is now **Juice Night Out** — named for and modeled on the Juice Night Out label sound, the
same way those parties themselves were themed around the label's music. The other selectas (House,
Battle, Open Format, Warehouse Rave, Jungle) have been pulled from the menu so the focus is entirely
on maxing out one deeply-conditioned profile ahead of the Party Grid v2 work. The genre engine
underneath is unchanged; only the selecta layer narrowed. Internal naming was also cleaned up so the
field is called "selecta" throughout, and the selecta picker's tooltip now reads simply "pick the
selecta that matches your vibe."

## V1.5.0 — Glover Light-Show, FX Voice & Party Grid

The biggest release yet — Chippy grew from a generator into a full performance / light-show instrument.

**The GLOVER light-show** (renamed from RAVE ON) — a full-screen visual with six selectable tricks,
each its own distinct look behind the always-present Chippy family: Tracers (comets with trails),
Liquid (flowing water), Tutting (LED grid + geometric line), Strobe (rays + flashing beams), Orbits
(spirograph), Bloom (an LED wall of blooming color).

**The FX voice (module 8)** — a dedicated effects voice with seven types: Riser Up/Down, Glitch
Up/Down (chopped stutter), Impact (stab), Sweep, and Ship Horn (the cheesy foghorn). Plus a per-voice
reverb button on every module, and a Master FX bus (reverb on the whole mix except drums) with a
strength control.

**The Party Grid** — the automation is now a clean constraint matrix: the selecta owns bar-length and
the record crate, the genre owns tempo, the time slot owns level/energy/pace. New selectas: Jungle
(pegged to D&B) and Juice Night Out (funky/tech house). Party opens correctly per the selecta and evolves
live in the UI.

**Selecta profiles (DNA)** — selectas can now carry a profile derived from AMU analysis of real
reference tracks. Juice Night Out was built from analyzing 12 of Wes's own 124–128 house tracks — the
first AMU-informed, catalog-modeled generative DJ profile.

**Plus** — the whole UI went modular (Eurorack-style boxes everywhere), the About tab was rebuilt, a
random title generator names each loop from classed rave-culture word banks, and a lot of polish.
