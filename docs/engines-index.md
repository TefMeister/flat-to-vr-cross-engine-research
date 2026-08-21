# Engine quick-lookup index

Find your engine, its typical render API, and the known public VR path. This is an orientation
aid, not an exhaustive database — contributions welcome (see
[`../CONTRIBUTING.md`](../CONTRIBUTING.md)). Every tool named is credited in
[`../ATTRIBUTION.md`](../ATTRIBUTION.md).

| Engine | Typical render API | Known public VR path | Notes |
|--------|--------------------|----------------------|-------|
| **Unreal Engine 4.8 – 5.x** | D3D11/D3D12 | **UEVR** (turnkey injector) | Drives UE's native stereo via UObject/FName reflection. Best-supported case. |
| **Unreal Engine 2 / 3** | D3D8 (UE2) / D3D9 (UE3) | None turnkey | Below UEVR's floor. UE3 is UE4's ancestor → UEVR is conceptual reference only. Manual build. |
| **Capcom RE Engine** | D3D11/D3D12 (+Vulkan) | **REFramework VR** (turnkey) | Engine ships an OpenVR path; REFramework activates it. |
| **Bethesda Creation Engine 2** (Starfield) | D3D12 | **starfield2vr** (mutars) | Public adapter; Reflex-marker frame timing. See [case study](./case-studies/creation-engine-2.md). |
| **Ubisoft AnvilNext 2.0** (Assassin's Creed) | D3D12 | **anvilengine2vr** (mutars) | Public multi-title adapter; two-hook frame timing. See [case study](./case-studies/anvil-per-eye-camera.md). |
| **Unity** | D3D11/12 (Vulkan/GL) | **UUVR** (universal) / per-game BepInEx plugins | Managed C# + built-in XR = easiest big engine class. Mono easier than IL2CPP. See [Unity games](./unity/). |
| **Valve Source** (SDK 2013 titles) | D3D9 | **HL2VR** (SDK mod) / per-game (L4D2VR) | Source SDK 2013 = near-source access for its titles. See [source-available](./source-available/). |
| **GoldSrc** (Half-Life 1) | OpenGL | **Lambda1VR** (via Xash3D-FWGS) | Open reimplementation makes it source-port territory. |
| **id Tech 1–4 & kin** (Doom, Quake 1–3, Doom 3, RTCW, Jedi Knight) | OpenGL | **Source-port VR conversions** (GZ3Doom/QuestZDoom, Quake VR, dhewm3-based, Team Beef ports) | GPL source releases — VR is built inside the engine. See [source-available](./source-available/). |
| **id Tech 5** (STEM/Evil Within) | D3D11 | None turnkey | 64-bit D3D11; strong candidate for a new adapter. Typically Z-up basis. Per-draw MVP. Source NOT released (unlike id Tech 1–4). |
| **Ubisoft Dunia** (Far Cry 2) | D3D9 | vorpX (generic) for 3D | Older D3D9; manual for true 6DoF. |
| **Bespoke / older custom engines** | D3D9 and older | Case-by-case | Usually fully manual; vorpX/geo-11 for seated 3D if D3D9+. |
| **Anything Direct3D 8 or older** | D3D8/7 | Wrapper first | Needs a D3D8→9/11 shim before modern stereo tooling applies. |

## How to identify an unknown engine (static, no launch needed)

- **PE header** → 32- vs 64-bit (Machine field: `0x14c` = x86, `0x8664` = x64).
- **Imported/embedded strings** → render API (`d3d8/9/11/12`, `dxgi`, `vulkan`, `opengl32`),
  VR hints (`openvr`, `openxr`), DRM (`steam_api`, Denuvo), middleware (PhysX, Bink),
  engine tags (`UnrealEngine3`, `idTech`, engine-specific renderer names).
- **Folder layout** → Unreal (`Engine/`, `<Game>Game/`, `.u`/`.pak`), id Tech (`base/`,
  virtual textures), etc.
- **A tiny main exe** often means a launcher stub; the real renderer lives in a companion DLL —
  fingerprint that instead.

A native **`openvr`/`openxr`** string in the main binary is a strong signal the engine already
has a VR path worth activating (the RE Engine case).

## Legend

- **Source port / SDK mod** = engine source is public; VR is built inside it — see
  [source-available conversions](./source-available/).
- **Turnkey** = an existing public tool you can run; you write little or no code.
- **Manual build** = use the [engine-agnostic core](./engine-agnostic-core/) +
  [porting checklist](./porting/) to write a new adapter.
- **Generic (vorpX/geo-11)** = seated stereoscopic-3D over an unmodified game; not full 6DoF.
  For the D3D9-specific route (and the dgVoodoo2 wrapper for D3D8/older), see
  [generic drivers for older D3D9 games](./generic-drivers/).

## Sources

Tools referenced above: [UEVR](https://github.com/praydog/UEVR) ·
[REFramework](https://github.com/praydog/REFramework) ·
[starfield2vr](https://github.com/mutars/starfield2vr) ·
[anvilengine2vr](https://github.com/mutars/anvilengine2vr) ·
[vorpX](https://www.vorpx.com/) ·
[geo-11](https://github.com/ThreeDeeJay/geo-11). Full credit list:
[`../ATTRIBUTION.md`](../ATTRIBUTION.md).
