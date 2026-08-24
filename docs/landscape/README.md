# The flat→VR tool landscape

The public tools for putting a flat game into VR fall into **seven distinct families**. They are
**not** interchangeable — picking the right family for your engine is the first decision. All
projects named here are credited in [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).

| Family | Examples | What it does | True 6DoF stereo? |
|--------|----------|--------------|-------------------|
| **Source ports / SDK mods** | **Team Beef** Quest ports, **Quake VR**, **HL2VR**, **GZ3Doom** | VR built *inside* an open/GPL/SDK engine | **Yes** — the gold standard |
| **Engine-native injectors** | **UEVR** (Unreal 4.8–5.x), **REFramework VR** (RE Engine) | Drives the engine's *own* stereo pipeline and object graph | **Yes** — best quality |
| **Managed-runtime mods (Unity)** | **UUVR**, **VRGIN**, **NomaiVR**, **TwoForksVR** | Loads C# plugins that switch on Unity's built-in XR support | **Yes** (per-game work for hands) |
| **Per-game native mods** | **Luke Ross R.E.A.L.** (GTA V, Cyberpunk, RDR2, HZD) | Bespoke per title; uses **AER** where no native stereo exists | Yes (with AER caveats) |
| **Generic VR drivers** | **vorpX** (D3D9–11) | Reconstructs a VR view over an unmodified game (geometry-3D / Z-buffer) | Partial — not engine-true |
| **Stereo-3D shader drivers** | **geo-11 / Helix**, **3Dmigoto**, **SuperDepth3D** (ReShade) | Stereoscopic 3D via shader injection; per-game fixes | 3D only — no 6DoF/runtime |
| **Performance kits** (not VR-adders) | **vrperfkit**, **OpenXR Toolkit** | Foveated / VRS upscaling for games that are *already* VR | n/a |

## How to read the table

- **Source ports / SDK mods** apply only where engine source is public (GPL id Tech releases,
  GoldSrc reimplementations, Source SDK 2013, GZDoom…) — but where they apply, nothing else
  competes: VR is built inside the engine. See
  [source-available conversions](../source-available/).
- **Engine-native injectors** give the best result but only exist for engines someone has
  deeply reverse-engineered and that expose (or can be made to expose) a stereo path. If one
  exists for your engine, use it — don't reinvent it.
- **Per-game native mods** are what you build when no injector covers your engine. They are the
  domain of the [porting model](../porting/) and, for modern AAA engines that can't render the
  world twice per frame, the [AER technique](../per-game-native-mods/).
- **Generic drivers** (vorpX) are the pragmatic fallback for older D3D9–11 games: quick seated
  3D without engine work, but not true per-eye engine rendering and rarely full 6DoF.
- **Stereo-3D shader drivers** (geo-11, 3Dmigoto) produce stereoscopic depth, not a VR runtime
  experience — useful for 3D displays or as a building block, not a finished VR mod.
- **Performance kits** do not add VR; they make existing VR cheaper. Listed here only so they
  aren't mistaken for VR-enablers.

## Decision shortcut

0. **Engine source public** (GPL release, open reimplementation, vendor SDK)? → build or use a
   [source-port VR conversion](../source-available/). Done — best possible outcome.
1. **Unreal Engine 4.8–5.x?** → UEVR. Done.
2. **Capcom RE Engine?** → REFramework VR. Done.
2b. **Unity?** → UUVR / a BepInEx per-game plugin — see [Unity games](../unity/). Done.
3. **Another modern engine with a big modding scene?** → check whether a per-game native mod
   already exists (see [case studies](../case-studies/)); if so, study/extend it. If the engine
   can't draw the world twice per frame, [AER](../per-game-native-mods/) is the known strategy.
4. **Nothing exists, but the engine is D3D11/12 and 64-bit?** → best candidate to build a new
   adapter with the [porting checklist](../porting/).
5. **Old D3D9 (or older) game, no appetite to build a mod?** → try **vorpX** / **geo-11** for
   seated 3D — see the dedicated guide: [generic drivers for older D3D9 games](../generic-drivers/).
6. **Direct3D 8 or earlier?** → you likely need an API wrapper (e.g. a D3D8→9/11 shim like
   **dgVoodoo2**) before any modern stereo tool even applies. See
   [generic drivers](../generic-drivers/).

See [`../engines-index.md`](../engines-index.md) for a per-engine quick lookup.

## Where the scene lives

The hub of all of this is the **Flat2VR community** ("Flatscreen to VR Modding", 150k+ members
on Discord) — announcements, per-game channels, and most of the authors above. In 2024 it also
spawned **Flat2VR Studios**, which produces *officially licensed* VR ports of flat games with
developers hired from the modding scene — a sign of how far the hobby has matured. At the
**August 2026 VR Games Showcase** the studio announced licensed ports of **System Shock VR**,
**High On Life**, **Out of Sight**, **Surviving Mars**, **Postal 2**, and **Shadowgate VR: The
Mines of Mythrok** — the pipeline from "fan mod" to "official licensed port" keeps widening.
[flat2vrstudios.com](https://www.flat2vrstudios.com/) ·
[x.com/Flat2VR](https://x.com/Flat2VR) · [beacons.ai/flat2vr](https://beacons.ai/flat2vr)

## Sources

- UEVR — [github.com/praydog/UEVR](https://github.com/praydog/UEVR) · [docs.uevr.io](https://docs.uevr.io/) · [uevr.org](https://uevr.org/)
- REFramework — [github.com/praydog/REFramework](https://github.com/praydog/REFramework) · [reframework.dev](https://reframework.dev/)
- Luke Ross R.E.A.L. / AER — framework now free (donations) since Mar 2026, Cyberpunk 2077 excluded; GTA V repo unlicensed (view-only); technique reference only — [github.com/LukeRoss00/gta5-real-mod](https://github.com/LukeRoss00/gta5-real-mod) · [patreon.com/realvr](https://www.patreon.com/realvr) · [Road to VR](https://roadtovr.com/luke-ross-vr-mods-free-cyberpunk-2077/)
- vorpX — commercial, closed source (public prior art only) — [vorpx.com](https://www.vorpx.com/)
- geo-11 / Helix — [github.com/ThreeDeeJay/geo-11](https://github.com/ThreeDeeJay/geo-11) · [helixmod.blogspot.com](https://helixmod.blogspot.com/)
- 3Dmigoto — [github.com/bo3b/3Dmigoto](https://github.com/bo3b/3Dmigoto) · [3dmigoto.com](https://www.3dmigoto.com/)
- vrperfkit — [github.com/fholger/vrperfkit](https://github.com/fholger/vrperfkit)
- Team Beef — [teambeefvr.com](https://www.teambeefvr.com/) · UUVR/VRGIN/Quake VR/HL2VR — see
  [source-available](../source-available/), [unity](../unity/), and
  [runtime layers](../runtime-layers/) for per-project links
- Flat2VR community & Studios — [flat2vrstudios.com](https://www.flat2vrstudios.com/)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
