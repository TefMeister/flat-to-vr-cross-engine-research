# Attribution & sources

Everything in this library is derived from **publicly available** work by the people and
projects below. **They own it; we only organize and link to it.** This page is the master
credit list — every doc in the repo points back here.

**We learn the techniques, not the code.** We do not copy, host, or reuse anyone's actual source
code or files — including mutars' (starfield2vr / anvilengine2vr, MIT) and Luke Ross's (R.E.A.L.)
work — **even where the license permits it and even where the work is now free to download.** We
describe the public *idea* in our own words and link back to the author's implementation, which
stays theirs.

**Corrections & removals — no questions asked.** If you are credited or referenced anywhere here
and want your name or information changed or removed, email **td3kxlvr@proton.me** and tell us
**exactly what to remove** — we will remove it promptly, no problem and no argument. We credit
everyone who helped, published, or even just inspired this work — tools, open-source projects,
community knowledge, and individuals alike; if we missed you, that's a mistake to fix, so tell us.

---

## People & primary projects

- **praydog** — **UEVR** (Universal Unreal Engine VR Injector) and **REFramework** (RE Engine
  mod loader, scripting platform, and VR). The foundation of most modern flat→VR modding; the
  reference for the Unreal case and the upstream of most engine-agnostic cores below.
  - UEVR: <https://github.com/praydog/UEVR> · docs: <http://docs.uevr.io/>
  - REFramework: <https://github.com/praydog/REFramework> · <https://reframework.dev/>
- **Elliott Tate** (github.com/elliotttate) — **vrframework** (**MIT**, a REFramework derivative):
  an engine-agnostic VR core plus a 17-part porting field guide. Source of the `IEngineAdapter`
  model and the 10-milestone porting checklist described here (concept only — we use no code).
  - <https://github.com/elliotttate/vrframework>
- **mutars — Sergii Permiakov** (github.com/mutars) — **starfield2vr** (Bethesda Creation
  Engine 2) and **anvilengine2vr** (Ubisoft AnvilNext 2.0), both **MIT**: the non-Unreal reference
  adapters that prove the pattern generalizes. Source of the Reflex-marker vs two-hook frame-timing
  contrast, the per-eye TAA-history fix, and the multi-title adapter structure — all documented as
  *technique*, with **no code copied from these repos**. Also `Geo3D` (**BSD-2-Clause**),
  `stalker2-uevr` (**unlicensed / all rights reserved**), and maintenance of `CommonLibSF`
  (GPL-3.0).
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

- **Raicuparta** (github.com/Raicuparta) — **UUVR** (universal Unity VR mod, **GPL-3.0**),
  **Rai Pal** (mod manager for universal mods, **GPL-3.0**), **NomaiVR** (Outer Wilds VR, with
  **artumino**, **MIT**), **TwoForksVR** (Firewatch VR, **MIT**). The reference body of work for
  Unity flat→VR modding.
  - <https://github.com/Raicuparta/uuvr> · <https://github.com/Raicuparta/rai-pal> ·
    <https://raicuparta.com/>
- **Eusth** — **VRGIN** (**MIT**), the classic VR injection framework for Unity games; its
  "Hacking VR into a Unity game" wiki remains a foundational public explainer.
  - <https://github.com/Eusth/VRGIN>
- **Team Beef** — **DrBeef**, **Baggyg**, and contributors (incl. **emileb**'s multithreaded
  GLES rendering) — the standalone-headset source-port VR conversions: QuestZDoom (**GPL-3.0**),
  Lambda1VR (**GPL-3.0**), Doom3Quest, RTCWQuest (**LGPL-3.0**), JKXR (**GPL-2.0**),
  Quake/Quake 3 ports, and more.
  - <https://www.teambeefvr.com/> · <https://github.com/Team-Beef-Studios> ·
    <https://www.patreon.com/teambeef>
- **Vittorio Romeo** (github.com/vittorioromeo) — **Quake VR** (**GPL-2.0**, roomscale +
  hand-interaction showcase) and **HL2VRU** (Half-Life 2: VR Mod — Unleashed fork).
  - <https://github.com/vittorioromeo/quakevr> · <https://github.com/vittorioromeo/HL2VRU>
- **Christopher M. Bruns (cmbruns)** — **GZ3Doom** (**GPL-3.0**), the original classic-Doom-in-VR
  GZDoom fork; continued by **Fishbiter** and **hh79** (gzdoomvr).
  - <https://github.com/Fishbiter/gz3doom> · <https://github.com/hh79/gzdoomvr>
- **KozGit** — **DOOM 3 BFG VR: Fully Possessed** (**GPL-3.0**), the PCVR Doom 3 conversion whose
  VR gameplay layer informed later ports.
  - <https://github.com/KozGit/DOOM-3-BFG-VR>
- **Source VR Mod Team** — **Half-Life 2: VR Mod** (free on Steam, built on Source SDK 2013).
  - <https://halflife2vr.com/>
- **sd805** — **L4D2VR** (Left 4 Dead 2 VR mod, no license file — all rights reserved by
  default; referenced as public prior art only).
  - <https://github.com/sd805/l4d2vr>
- **Gistix** (Giovanni Correia) and the **Portal4Dead** team — **Portal2VR** (Portal 2 VR mod,
  released ~December 2025; binary D3D9 injection built on sd805's L4D2VR pattern plus DXVK and
  Source SDK 2013 material; no license file — all rights reserved by default; referenced as
  public prior art only).
  - <https://github.com/Gistix/portal2vr> · <https://www.portal4dead.com/>

## Tools, drivers & communities

- **vorpX** (Ralf Ostertag) — generic D3D9–11 VR driver. **Commercial, closed source**;
  referenced as public prior art only (no code inspected or reused). <https://www.vorpx.com/>
- **Vireio Perception** (cybereality and contributors) — free, **open-source (LGPL-3.0)** generic
  VR driver in the same family as vorpX, publicly available since 2013. Referenced as public prior
  art / an open alternative worth trying before reaching for paid software.
  <https://github.com/cybereality/Perception>
- **geo-11 / Helix Mod** (incl. ThreeDeeJay, Helifax, DarkStarSword and the 3D-fix community) —
  stereoscopic-3D D3D11 driver and per-game fixes.
  <https://github.com/ThreeDeeJay/geo-11> · <https://helixmod.blogspot.com/>
- **3Dmigoto** (bo3b, DarkStarSword, DHR, contributors) — D3D11 shader-injection modding tool.
  <https://github.com/bo3b/3Dmigoto> · <https://www.3dmigoto.com/>
- **vrperfkit** (fholger) — VR performance toolkit (foveated / VRS upscaling).
  <https://github.com/fholger/vrperfkit>
- **fholger** — also author of **farcry_vrmod** (added 2026-08-24), a full roomscale 6DoF VR
  conversion of the original **Far Cry (2004, Crytek)** built against the official **CryEngine
  Mod SDK** — a vendor-SDK case in the same family as Source SDK 2013, referenced here as
  technique/landscape only, no code reused. <https://github.com/fholger/farcry_vrmod>
- **dgVoodoo2** (dege-diosg) — wraps legacy DirectX (Glide/DX1–9) onto D3D11/12; the bridge that
  lets D3D9-and-older games reach geo-11 / 3Dmigoto. <https://github.com/dege-diosg/dgVoodoo2>
- **ViGEmBus** (Nefarius) — virtual gamepad driver used by some adapters for controller
  emulation. <https://github.com/nefarius/ViGEmBus>
- **ZNix** and the OpenComposite community forks — **OpenComposite / OpenOVR** (**GPL-3.0**),
  the OpenVR→OpenXR reimplementation. <https://gitlab.com/znixian/OpenOVR>
- **Matthieu Bucchianeri (mbucchia)** — **OpenXR Toolkit** (**MIT**; development ended,
  still widely used). <https://mbucchia.github.io/OpenXR-Toolkit/>
- **oneup03** — **VRto3D** (**LGPL-3.0**), the SteamVR virtual-headset driver for 3D displays.
  <https://github.com/oneup03/VRto3D>
- **BlueSkyDefender** — **Depth3D / SuperDepth3D** (no repo-level license; see file headers),
  the ReShade depth-buffer stereo shader. <https://github.com/BlueSkyDefender/Depth3D>
- **crosire** and contributors — **ReShade**, the post-processing injector SuperDepth3D runs on.
  <https://reshade.me/>
- **BepInEx team** (<https://github.com/BepInEx/BepInEx>), **MelonLoader / LavaGang**
  (<https://github.com/LavaGang/MelonLoader>), and **Andreas Pardeike** — **Harmony**
  (<https://github.com/pardeike/Harmony>) — the managed injection/patching substrate of Unity
  modding.
- Non-VR source ports the VR conversions build on: **GZDoom** (ZDoom team), **dhewm3**
  (<https://github.com/dhewm/dhewm3>), **ioquake3**, **Xash3D-FWGS** (FWGS team,
  <https://github.com/FWGS/xash3d-fwgs>) — and **id Software** for releasing its engines under
  the GPL in the first place, plus **Valve** for the Source SDK.
- **Flat2VR community & Flat2VR Studios** — the flatscreen-to-VR hub (150k+ member Discord) and
  the studio producing officially licensed VR ports with developers from the modding scene. At the
  **August 2026 VR Games Showcase** the studio announced official VR ports of **System Shock VR**
  and **High On Life**, alongside **Out of Sight**, **Surviving Mars**, **Postal 2**, and
  **Shadowgate VR: The Mines of Mythrok** (PSVR2) — evidence the licensed-port pipeline keeps
  scaling. <https://www.flat2vrstudios.com/>
  - **Impact Reality** — Flat2VR Studios' parent company; confirmed (Aug 2026) $5M+ raised in a
    private round (Hartmann Capital among backers) plus a **StartEngine** equity-crowdfunding
    reservation opened to retail investors. Portfolio named in that campaign: Trombone Champ:
    Unflattened, WRATH: Aeon of Ruin VR, FlatOut 4: Total Insanity VR, Roboquest VR, Surviving
    Mars: Pioneer, VRacer Hoverbike, POSTAL 2, Primal Rumble, I Am Your Beast, Out of Sight, R.A.I.D.,
    High On Life VR, System Shock VR. Referenced as landscape/funding context only — no code or
    product of theirs is used here.
  - **Impact Inked** — a sibling publishing label; announced (Aug 2026) a PSVR2 publishing
    partnership including **Drop Dead: The Cabin**.
- **Camracks** — **PCVR Central**, a community-run, non-rehosting directory of PC VR mods and
  conversions (added 2026-08-24; ~968 flat games tracked, ~900 mods cataloged with quality/
  freshness labels, a Steam-library VR-compatibility checker, and per-mod links back to each
  creator's own page). Useful as a landscape/discovery cross-check alongside this library.
  <https://pcvrcentral.com/>
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
