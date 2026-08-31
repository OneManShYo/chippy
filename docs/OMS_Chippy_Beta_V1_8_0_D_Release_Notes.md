# OMS Chippy [Beta V1_8_0] — RELEASE NOTES

**Live at chippy.onemanshyo.com**

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
