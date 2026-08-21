# The flat→VR tool landscape

The public tools for putting a flat game into VR fall into **five distinct families**. They are
**not** interchangeable — picking the right family for your engine is the first decision. All
projects named here are credited in [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).

| Family | Examples | What it does | True 6DoF stereo? |
|--------|----------|--------------|-------------------|
| **Engine-native injectors** | **UEVR** (Unreal 4.8–5.x), **REFramework VR** (RE Engine) | Drives the engine's *own* stereo pipeline and object graph | **Yes** — best quality |
| **Per-game native mods** | **Luke Ross R.E.A.L.** (GTA V, Cyberpunk, RDR2, HZD) | Bespoke per title; uses **AER** where no native stereo exists | Yes (with AER caveats) |
| **Generic VR drivers** | **vorpX** (D3D9–11) | Reconstructs a VR view over an unmodified game (geometry-3D / Z-buffer) | Partial — not engine-true |
| **Stereo-3D shader drivers** | **geo-11 / Helix**, **3Dmigoto** | Stereoscopic 3D via shader injection (mostly D3D11); per-game fixes | 3D only — no 6DoF/runtime |
| **Performance kits** (not VR-adders) | **vrperfkit** | Foveated / VRS upscaling for games that are *already* VR | n/a |

## How to read the table

- **Engine-native injectors** give the best result but only exist for engines someone has
  deeply reverse-engineered and that expose (or can be made to expose) a stereo path. If one
  exists for your engine, use it — don't reinvent it.
- **Per-game native mods** are what you build when no injector covers your engine. They are the
  domain of the [porting model](../porting/) in this library.
- **Generic drivers** (vorpX) are the pragmatic fallback for older D3D9–11 games: quick seated
  3D without engine work, but not true per-eye engine rendering and rarely full 6DoF.
- **Stereo-3D shader drivers** (geo-11, 3Dmigoto) produce stereoscopic depth, not a VR runtime
  experience — useful for 3D displays or as a building block, not a finished VR mod.
- **Performance kits** do not add VR; they make existing VR cheaper. Listed here only so they
  aren't mistaken for VR-enablers.

## Decision shortcut

1. **Unreal Engine 4.8–5.x?** → UEVR. Done.
2. **Capcom RE Engine?** → REFramework VR. Done.
3. **Another modern engine with a big modding scene?** → check whether a per-game native mod
   already exists (see [case studies](../case-studies/)); if so, study/extend it.
4. **Nothing exists, but the engine is D3D11/12 and 64-bit?** → best candidate to build a new
   adapter with the [porting checklist](../porting/).
5. **Old D3D9 (or older) game, no appetite to build a mod?** → try **vorpX** / **geo-11** for
   seated 3D.
6. **Direct3D 8 or earlier?** → you likely need an API wrapper (e.g. a D3D8→9/11 shim) before any
   modern stereo tool even applies.

See [`../engines-index.md`](../engines-index.md) for a per-engine quick lookup.

## Sources

- UEVR — [github.com/praydog/UEVR](https://github.com/praydog/UEVR) · [docs.uevr.io](https://docs.uevr.io/) · [uevr.org](https://uevr.org/)
- REFramework — [github.com/praydog/REFramework](https://github.com/praydog/REFramework) · [reframework.dev](https://reframework.dev/)
- Luke Ross R.E.A.L. / AER — [github.com/LukeRoss00/gta5-real-mod](https://github.com/LukeRoss00/gta5-real-mod) · [patreon.com/realvr](https://www.patreon.com/realvr)
- vorpX — [vorpx.com](https://www.vorpx.com/)
- geo-11 / Helix — [github.com/ThreeDeeJay/geo-11](https://github.com/ThreeDeeJay/geo-11) · [helixmod.blogspot.com](https://helixmod.blogspot.com/)
- 3Dmigoto — [github.com/bo3b/3Dmigoto](https://github.com/bo3b/3Dmigoto) · [3dmigoto.com](https://www.3dmigoto.com/)
- vrperfkit — [github.com/fholger/vrperfkit](https://github.com/fholger/vrperfkit)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
