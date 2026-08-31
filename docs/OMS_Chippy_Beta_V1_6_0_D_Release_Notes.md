# OMS Chippy [Beta V1_6_0] — RELEASE NOTES

**Live at chippy.onemanshyo.com**

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
