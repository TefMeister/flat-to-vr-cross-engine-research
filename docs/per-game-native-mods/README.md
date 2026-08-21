# Per-game native VR mods & AER — for modern AAA engines

Some modern AAA engines have no universal injector (they aren't Unreal, RE Engine, etc.) and
cannot draw the world twice per frame fast enough for true simultaneous stereo. The public answer
for these has been the **per-game native mod** built around **AER (Alternating Eye Rendering)**,
the technique popularized by **Luke Ross's R.E.A.L. mods**.

This page describes **only the publicly-documented technique**, as an educational reference. See
the property note below.

> ### Property & scope note (read this)
> **AER is a technique; Luke Ross's R.E.A.L. mods are his own property.** This library does not
> include, host, redistribute, reproduce, or use any of his code or mods. It documents the
> *publicly-explained concept* of alternating-eye rendering and links to his official channels.
> If you want to use a R.E.A.L. mod, get it from his official Patreon and support the author. As
> with everything in this repo: we use nothing that anyone claims as their property, and we honor
> correction/removal requests — email **td3kxlvr@proton.me**.

---

## The problem AER solves

True VR wants the scene drawn **twice per frame** — one camera per eye, at the same instant — at
a high framerate (e.g. 2 × 90 = 180 fps at VR resolution/FOV). Heavy modern engines (RAGE,
RED Engine, Decima, and similar closed AAA renderers) simply **cannot** render the full world
twice per tick at those rates. This is the same "closed engine renders once per frame" wall the
[porting checklist](../porting/) hits at its AFR milestone.

## What AER does (the public concept)

Instead of two full renders per frame, **AER draws one eye per engine tick and reprojects the
other eye from the previous frame:**

```
tick 1: render L
tick 2: render R,  reproject L from tick 1
tick 3: render L,  reproject R from tick 2
...
```

So each eye is *natively* rendered every other frame and *reprojected* on the frames between —
roughly halving the rendering cost versus full simultaneous stereo. Additional public techniques
described alongside it:

- **Camera-rotation compensation / "yaw folding"** — corrects the reprojected eye for head/camera
  rotation so turning stays smooth rather than smearing.
- The known trade-off is **ghosting** on the reprojected eye; the publicly-announced **AER v2**
  (2023) reduced it substantially, and later updates added modern upscalers (DLSS) for sharper
  results.

This makes AER the go-to strategy precisely where native stereo and even same-tick
"synchronized sequential" rendering are impossible — see how it sits among the other
[stereo submission strategies](../techniques/#stereo-submission-strategies).

## How it differs from the injectors in this library

| | Universal injector (UEVR, REFramework) | Per-game native mod (R.E.A.L. + AER) |
|---|---|---|
| Coverage | Every game on a supported engine | One specific title at a time |
| Mechanism | Drives the engine's own/near-native stereo | Bespoke hooks + AER reprojection |
| Stereo | Native or same-tick sequential where possible | Alternating-eye + reprojection |
| Distribution | Open-source frameworks | Author's own mods — **free (donations) since March 2026**, Cyberpunk 2077 excluded; the GTA V repo is source-available but unlicensed (viewable, not reusable) |

Titles the R.E.A.L. mods have publicly supported include Cyberpunk 2077, Elden Ring, Hogwarts
Legacy, Horizon Zero Dawn (1 & 2), Marvel's Spider-Man, Uncharted 4, Red Dead Redemption 2, and
GTA V — around 31 titles in total — as a sense of which engine classes AER has been applied to.
(List per public reporting; availability and specifics are the author's to state.)

**Availability, verified against public reporting (Aug 2026):** as of **15 March 2026** the
R.E.A.L. framework is distributed **free for everyone with donations encouraged** (it was
previously a paid Patreon), a change Ross made after **CD Projekt issued a DMCA takedown in
January 2026** over the paywalled Cyberpunk 2077 mod; **Cyberpunk 2077 is excluded** from the free
release. Separately, the **GTA V, Red Dead Redemption 2, and Mafia** VR mods were earlier taken
down after a **Take-Two legal complaint** — though the GTA V source repo is currently public again
(unlicensed). None of this changes our position: we use the *public technique*, not his code or
mods.

## What a project can *learn* from this (not take)

- When your target engine can't render the world twice per tick, **AER-style alternate-eye +
  reprojection is a legitimate, documented strategy** to reach a playable framerate — you would
  implement your own version, not lift anyone's code.
- Budget for **reprojection artifacts** (ghosting) and plan a rotation-compensation step.
- It is a **per-title** effort by nature — like the [case studies](../case-studies/), the reusable
  part is the *idea and the architecture*, not any specific implementation.

## Sources

- **Luke Ross — R.E.A.L. VR** (official) — [patreon.com/realvr](https://www.patreon.com/realvr)
- **GTA V R.E.A.L. mod** (the publicly-open one, unlicensed) —
  [github.com/LukeRoss00/gta5-real-mod](https://github.com/LukeRoss00/gta5-real-mod)
- Free-release + Cyberpunk exclusion — [Road to VR: "Luke Ross Releases PC VR Mod Suite for Free, Excluding Contentious 'Cyberpunk 2077' Mod"](https://roadtovr.com/luke-ross-vr-mods-free-cyberpunk-2077/)
- Take-Two takedown of GTA V / RDR2 / Mafia mods — [Road to VR](https://www.roadtovr.com/luke-ross-vr-mods-gta-red-dead-take-two-dmca-notice/)
- CD Projekt DMCA of the paid Cyberpunk 2077 mod — [Notebookcheck](https://www.notebookcheck.net/Paid-Cyberpunk-2077-VR-mod-targeted-by-CD-Projekt-Red-in-DMCA-takedown.1207067.0.html)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
