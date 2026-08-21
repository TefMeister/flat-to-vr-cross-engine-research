# Attribution & sources

Everything in this library is derived from **publicly available** work by the people and
projects below. **They own it; we only organize and link to it.** This page is the master
credit list — every doc in the repo points back here.

**If you should be credited and aren't, or you are a rights holder who wants a correction or
removal, email td3kxlvr@proton.me and it will be fixed as soon as possible.** We credit everyone
who helped, published, or even just inspired this work — tools, open-source projects, community
knowledge, and individuals alike.

---

## People & primary projects

- **praydog** — **UEVR** (Universal Unreal Engine VR Injector) and **REFramework** (RE Engine
  mod loader, scripting platform, and VR). The foundation of most modern flat→VR modding; the
  reference for the Unreal case and the upstream of most engine-agnostic cores below.
  - UEVR: <https://github.com/praydog/UEVR> · docs: <http://docs.uevr.io/>
  - REFramework: <https://github.com/praydog/REFramework> · <https://reframework.dev/>
- **Elliott Tate** (github.com/elliotttate) — **vrframework**: an engine-agnostic VR core plus a
  17-part porting field guide, derived from REFramework (MIT). Source of the `IEngineAdapter`
  model and the 10-milestone porting checklist described here.
  - <https://github.com/elliotttate/vrframework>
- **mutars — Sergii Permiakov** (github.com/mutars) — **starfield2vr** (Bethesda Creation
  Engine 2) and **anvilengine2vr** (Ubisoft AnvilNext 2.0): the non-Unreal reference adapters
  that prove the pattern generalizes. Source of the Reflex-marker vs two-hook frame-timing
  contrast, the per-eye TAA-history fix, and the multi-title adapter structure. Also `Geo3D`,
  `stalker2-uevr`, and maintenance of `CommonLibSF`.
  - <https://github.com/mutars/starfield2vr> · <https://github.com/mutars/anvilengine2vr>
- **Luke Ross** (github.com/LukeRoss00) — **R.E.A.L.** VR mods and the **AER (Alternating Eye
  Rendering)** technique for engines that cannot draw the world twice per frame. We reference the
  publicly-known *technique* only. Note the GTA V repo is source-**available** but **unlicensed**
  (all rights reserved by default — viewable, **not** reusable). Per public reporting (verified
  Aug 2026), the framework has been **free with optional donations since 15 March 2026** (formerly
  paid Patreon), with **Cyberpunk 2077 excluded** after a CD Projekt DMCA; the GTA V / RDR2 / Mafia
  mods were earlier removed following a Take-Two complaint. We copy no code from the repo and use
  no paid or patron-only material.
  - <https://github.com/LukeRoss00/gta5-real-mod> · <https://www.patreon.com/realvr> · [Road to VR (free release)](https://roadtovr.com/luke-ross-vr-mods-free-cyberpunk-2077/)

## Tools, drivers & communities

- **vorpX** (Ralf Ostertag) — generic D3D9–11 VR driver. **Commercial, closed source**;
  referenced as public prior art only (no code inspected or reused). <https://www.vorpx.com/>
- **geo-11 / Helix Mod** (incl. ThreeDeeJay, Helifax, DarkStarSword and the 3D-fix community) —
  stereoscopic-3D D3D11 driver and per-game fixes.
  <https://github.com/ThreeDeeJay/geo-11> · <https://helixmod.blogspot.com/>
- **3Dmigoto** (bo3b, DarkStarSword, DHR, contributors) — D3D11 shader-injection modding tool.
  <https://github.com/bo3b/3Dmigoto> · <https://www.3dmigoto.com/>
- **vrperfkit** (fholger) — VR performance toolkit (foveated / VRS upscaling).
  <https://github.com/fholger/vrperfkit>
- **dgVoodoo2** (dege-diosg) — wraps legacy DirectX (Glide/DX1–9) onto D3D11/12; the bridge that
  lets D3D9-and-older games reach geo-11 / 3Dmigoto. <https://github.com/dege-diosg/dgVoodoo2>
- **ViGEmBus** (Nefarius) — virtual gamepad driver used by some adapters for controller
  emulation. <https://github.com/nefarius/ViGEmBus>
- The **flatscreen-to-VR modding community** broadly — forums, wikis, Discord/Reddit threads,
  and countless per-game fix authors whose collected knowledge underlies all of the above.

## Open-source libraries commonly used by these projects

Each under its own license:

- **MinHook** (TsudaKageyu) — <https://github.com/TsudaKageyu/minhook>
- **safetyhook** (cursey) — <https://github.com/cursey/safetyhook>
- **Dear ImGui** (ocornut) — <https://github.com/ocornut/imgui>
- **GLM** (g-truc) — <https://github.com/g-truc/glm>
- **spdlog** (gabime) — <https://github.com/gabime/spdlog>
- **nlohmann/json** — <https://github.com/nlohmann/json>
- **OpenVR** (Valve) — <https://github.com/ValveSoftware/openvr>
- **OpenXR SDK** (Khronos) — <https://github.com/KhronosGroup/OpenXR-SDK>
- **DirectXTK12** (Microsoft) — <https://github.com/microsoft/DirectXTK12>
- **CommonLibSF** (Starfield Reverse Engineering) — **GPL-3.0** (copyleft; archived) —
  <https://github.com/Starfield-Reverse-Engineering/CommonLibSF>

Graphics debuggers named across these guides: **RenderDoc** (<https://renderdoc.org/>) and
Microsoft **PIX** (<https://devblogs.microsoft.com/pix/>).

## Engine / platform owners (context, not endorsement or affiliation)

Capcom (RE Engine) · Epic Games (Unreal Engine) · Bethesda/id Software (Creation Engine,
id Tech) · Ubisoft (AnvilNext, Dunia) · and the owners of every other engine referenced. This
library is unaffiliated with them; it redistributes none of their assets or source and takes no
position on their terms of use.

## Written / documentation sources consulted

UEVR docs (docs.uevr.io, uevr.org) · REFramework (reframework.dev) · UploadVR · DigiAlps ·
Luke Ross (patreon.com/realvr) · CompoundVR · VRDB · and the READMEs of every repo listed above.

---

*This library is non-commercial and educational. All trademarks and copyrights belong to their
respective owners.*
