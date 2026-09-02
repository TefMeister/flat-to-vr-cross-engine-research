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

- **Vice City VR** (github.com/dubrovskiy-yevhen-stakelogic, with a native-Quest sibling maintained
  by **Blackbird88**) — an unofficial stereoscopic 6DoF **OpenXR** conversion of the 2003 PC release
  of GTA: Vice City, and the only substantial VR prior art on the **RenderWare** family. Cited here
  for what its own public description establishes about *method*: it is built on a reverse-engineered
  source reimplementation of the game plus **librw**, and **replaces the graphics pipeline outright**
  (Direct3D 12, single-pass stereo, variable-rate-shading foveation, DLAA/FSR 2) rather than hooking
  the shipped renderer. Only the release is public; the runtime source is private during development.
  We have read its public description only — no download, no clone, no code. Noted for completeness:
  the underlying GTA III / Vice City reimplementation repository is subject to a publisher takedown
  and returns HTTP 451 on GitHub as of 2026-09-01.
  - <https://github.com/dubrovskiy-yevhen-stakelogic/vice-city-vr> · <https://github.com/Blackbird88/vice-city-vr-quest>
- **aap** (github.com/aap) — **librw** (**MIT**), an open-source reimplementation of the RenderWare
  graphics engine and the rendering foundation the project above depends on; actively maintained.
  Referenced as landscape context on the RenderWare family page; no code used.
  - <https://github.com/aap/librw>

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
- **alphazolam** — **EMV Engine**, the long-running RE Engine editor/inspector script suite for
  REFramework. Referenced as public precedent for prefab instantiation being the sanctioned way to
  spawn a complete object, and for its own documented caution about component-level spawning.
  Studied online only; no code reused. See the repo for its license.
  - <https://github.com/alphazolam/EMV-Engine>
- **Ekey** — **REE.PAK.Tool**, whose published per-title file lists let a game's shipped asset paths
  be enumerated offline without unpacking anything. Referenced for the file lists as a public index;
  no game content redistributed here.
  - <https://github.com/Ekey/REE.PAK.Tool>
- **godlock2000-eng** — **ResidentEvil2\_CustomAnimationFramework\_NonRTX**, whose documentation of
  the RE Engine motion / motion-list formats and runtime dynamic-motion-bank registration is the
  clearest public write-up of that subsystem we have found. Described in our own words; no code or
  files reused.
  - <https://github.com/godlock2000-eng/ResidentEvil2_CustomAnimationFramework_NonRTX>
- **Junh2x** — **RE9-Movement-Speed-Mod**, source of the pair-the-animation-speed-with-the-movement-
  driver technique for changing locomotion speed without foot-sliding.
  - <https://github.com/Junh2x/RE9-Movement-Speed-Mod>
- **Remleo** — [UEVR PR #433](https://github.com/praydog/UEVR/pull/433) (merged 2026-08-30), the fix
  for a gamma hook installing itself into a garbage vtable slot because an empty `std::optional`
  satisfied a `!= 0` test. Cited as one of the worked examples in
  [techniques → silent no-ops](./docs/techniques/README.md#silent-no-ops-verification-that-cannot-see-the-failure).
- **ErwinGunsmith** — [REFramework PR #1809](https://github.com/praydog/REFramework/pull/1809)
  (merged 2026-08-28), restoring `on_pre_gui_draw_element`'s `false` return after a nine-day silent
  regression introduced in [PR #1503](https://github.com/praydog/REFramework/pull/1503). Cited in the
  same section, and as a live warning on the
  [RE Engine family page](./docs/engines/re-engine.md).
- **prideslayer** and contributors — **VRIK Player Avatar** (Skyrim VR). Cited only to draw the
  distinction between the well-known VR floor-calibration/height-offset problem it addresses and the
  pose-dependent, animation-driven body float documented in
  [techniques](./docs/techniques/README.md#vr-body-height-the-hmd-anchored-float). No code or
  technique reused.
  - <https://www.nexusmods.com/skyrimspecialedition/mods/23416>

## Tools, drivers & communities

- **Fire-Head** — **MHNoDRM**, the community write-up documenting Manhunt (2003)'s 16 SecuROM-remnant
  IAT-hook addresses and their fake-return-value mechanism. Technique studied and independently
  verified against our own live memory dump; no code reused.
  <https://github.com/Fire-Head/MHNoDRM>
- **OldUnreal** (Smirftsch and the OldUnreal community) — custodians of Unreal Engine 1:
  the Epic-licensed 227 community maintenance patch line (currently 227k), with its SDK,
  64-bit builds, modern renderers, and render-device contract. The foundation of this account's
  Unreal Gold project; referenced as public documentation and a tool we actually run, no code
  reused.
  <https://github.com/OldUnreal> · <https://www.oldunreal.com/>
- **vorpX** (Ralf Ostertag) — generic D3D9–11 VR driver. **Commercial, closed source**;
  referenced as public prior art only (no code inspected or reused). <https://www.vorpx.com/>
- **Vireio Perception** (cybereality and contributors) — free, **open-source (LGPL-3.0)** generic
  VR driver in the same family as vorpX, publicly available since 2013. Referenced as public prior
  art / an open alternative worth trying before reaching for paid software.
  <https://github.com/cybereality/Perception>
- **geo-11 / Helix Mod** (incl. ThreeDeeJay, Helifax, DarkStarSword and the 3D-fix community) —
  stereoscopic-3D D3D11 driver and per-game fixes.
  <https://github.com/ThreeDeeJay/geo-11> · <https://helixmod.blogspot.com/>
- **Helifax — Octavian Vasilov** — **Vk3DVision**, a Vulkan stereoscopic-3D driver (and its
  predecessor **OGL3DVision** for OpenGL), with a maintained per-game fix list including DOOM (2016)
  and DOOM Eternal. **Closed source, Patreon-funded** — the public repo hosts compiled releases
  only; referenced here purely as public prior art and a feasibility proof that per-eye override at
  the Vulkan level works on modern closed commercial titles. No code inspected or reused.
  Also the author of a **6DoF VR mod for DOOM Eternal** (id Tech 7) using single-pass stereo
  instancing — distinct from the stereo-only Vk3DVision fixes, and cited here as technique prior art
  on that engine family. **Status noted 2026-09-01:** the Vk3DVision repository was archived by its
  owner on 2026-03-05 and is read-only, final release 4.25.5.
  <https://github.com/helifax/Vk3DVision-Public> · <https://3dsurroundgaming.com/Vk3DVisionGames.html>
- **emoose** (original author) and **brunoanc** (2024 update and re-host) — **DOOMLegacyMod**, which
  re-adds DOOM (2016)'s hidden console commands and cvars on the retail build without developer mode,
  and publishes `doom_cmds.txt` / `doom_cvars.txt` as plain-text interface dumps. **Closed source, no
  licence stated** — referenced here purely as public prior art, read online, never downloaded or
  studied as an implementation. It is what established, for this library, that a production-gated
  engine's cvars can be hidden-but-constructible rather than absent.
  <https://github.com/brunoanc/DOOMLegacyMod> · <https://github.com/emoose>
- **Flat2VR** — reporting that first documented Helifax's DOOM Eternal 6DoF work and its single-pass
  stereo-instancing approach, cited above.
  <https://x.com/Flat2VR/status/1704495949978984506>
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
  - **Flat2VR Spark** (announced August 2025) — the studio's programme under which modders build
    officially licensed adaptations from source, in small pods, with credit and revenue participation.
    Cited as landscape context only, from UploadVR's report.
    <https://www.uploadvr.com/flat2vr-spark-announcement/>
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
  conversions (added 2026-08-24; 991 flat games tracked as of 2026-08-31, up from ~967 four days
  earlier; ~900 mods cataloged with quality/freshness labels, a Steam-library VR-compatibility
  checker, and per-mod links back to each creator's own page). Useful as a landscape/discovery
  cross-check alongside this library.
  <https://pcvrcentral.com/>
- The **flatscreen-to-VR modding community** broadly — forums, wikis, Discord/Reddit threads,
  and countless per-game fix authors whose collected knowledge underlies all of the above.

## Open-source libraries commonly used by these projects

Each under its own license:

- **LZO** (Markus F.X.J. Oberhumer) — **GPL-2.0**, the compression library whose published decoder
  algorithm and constants (`lzo2a_d.ch`, `config2a.h`) were read online and independently
  transcribed to verify an LZO2A-compressed game archive byte-for-byte; no code copied into this
  library or into any project repo. <http://www.oberhumer.com/opensource/lzo/>
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
id Tech) · Ubisoft (AnvilNext, Dunia, Scimitar/Anvil) · Crytek (CryEngine) · Criterion
(RenderWare, Burnout) · Tango Gameworks (the STEM branch of id Tech 5) · Ninja Theory
(the NTEngine layer on Unreal Engine 3) · Avalanche Studios (Avalanche Engine) · Remedy
Entertainment (the Alan Wake engine) · Double Fine Productions (the Psychonauts engine) ·
and the owners of every other engine referenced. This
library is unaffiliated with them; it redistributes none of their assets or source and takes no
position on their terms of use.

## Written / documentation sources consulted

UEVR docs (docs.uevr.io, uevr.org) · REFramework (reframework.dev) · UploadVR · DigiAlps ·
Luke Ross (patreon.com/realvr) · CompoundVR · VRDB · and the READMEs of every repo listed above.

### Developer-authored engine talks & technical write-ups

- **Tiago Sousa** and **Jean Geffroy** (id Software) — *"The Devil is in the Details: idTech 666"*,
  SIGGRAPH 2016, Advances in Real-Time Rendering. The primary developer-authored description of
  id Tech 6's renderer.
  [slides](https://www.slideshare.net/TiagoAlexSousa/siggraph2016-the-devil-is-in-the-details-idtech-666) ·
  [text summary, 80.lv](https://80.lv/articles/idtech-666-the-secret-of-dooms-render)
- **Adrian Courrèges** — *"DOOM (2016) — Graphics Study"*, a frame-by-frame breakdown of the
  renderer's passes. <https://www.adriancourreges.com/blog/2016/09/09/doom-2016-graphics-study/>
- **Ubisoft** — *Dunia shader-pipeline architecture*, REAC 2023.
  <https://enginearchitecture.realtimerendering.com/downloads/reac2023_dunia_shader_pipeline.pdf>
- **id Software** — the official GPL source release of **Doom 3 BFG Edition**, used here as
  first-party documentation of how the id lineage applies stereo separation (`renderView_t`'s
  `vieworg`, `viewEyeBuffer` and `stereoScreenSeparation`) and of the `stereoRender_*` cvar family
  it later left dormant in id Tech 6. Read online as documentation; **no code taken.**
  <https://github.com/id-Software/DOOM-3-BFG>
- **NVIDIA** — published developer documentation for **3D Vision Automatic** and the **NVAPI stereo
  headers**, used here as the first-party description of the clip-space shader-footer mechanism
  (`ClipPos.x += Separation * (ClipPos.w - Convergence)`), the per-game-profile requirement, the
  post-processing/deferred-renderer caveat, and the Automatic-versus-Direct driver-mode distinction
  that tells a driver-stereo title apart from a natively stereo one. Read as documentation; no code
  taken.
  [background](https://archive.docs.nvidia.com/gameworks/content/technologies/desktop/nv3dva_background.htm) ·
  [stereoscopic issues](https://archive.docs.nvidia.com/gameworks/content/technologies/desktop/nv3dva_stereoscopic_issues.htm) ·
  [nvapi_lite_stereo.h](https://github.com/NVIDIA/nvapi/blob/main/nvapi_lite_stereo.h)
- **Microsoft** — the Direct3D 11 reference documentation on **Microsoft Learn**, used as the
  first-party statement that `ID3D11DeviceContext::ExecuteCommandList` returns `void` and may decline
  to execute a list on query-validation grounds, and that `RestoreContextState = FALSE` returns the
  context to its default state. Cited as vendor documentation of a silent-no-op hazard.
  [`ExecuteCommandList`](https://learn.microsoft.com/en-us/windows/desktop/api/D3D11/nf-d3d11-id3d11devicecontext-executecommandlist) ·
  [Command List overview](https://learn.microsoft.com/en-us/windows/win32/direct3d11/overviews-direct3d-11-render-multi-thread-command-list)
- **The HelixMod community**, including **Chiz** (Prince of Persia 2008 and Alice: Madness Returns
  fixes) — their published per-game stereoscopic-3D fix write-ups, changelogs and settings
  documentation, used here purely as **reports on engine behaviour**: what each fix had to correct,
  what it did not, and what its configuration implies about the game's camera paths. These fixes are
  closed-source or unlicensed; they were **read online only, never installed or copied**.
  <https://helixmod.blogspot.com/>
- **Epic Games** — UE3's own `Engine/Shaders/Common.usf`, as **shipped with a licensed retail game**,
  which documents the reserved vertex-shader registers (`ViewProjectionMatrix`, `CameraPosition`,
  `PreViewTranslation`) and their required agreement with the RHI's register enum. Read as
  documentation from a legitimately-owned installation; no engine source is redistributed here.

- **NVIDIA**, additionally — the published **`nvstereo.h` / `StereoParmsTexture` documentation**,
  used here as the first-party statement of the stereo-parameters texture's channel layout
  (per-eye separation, convergence, and the ±1 unit vector identifying the current eye), that
  `ParamTextureManager` is an application-side SDK helper (not a driver component) calling ordinary
  NVAPI queries, and the exact resource shape (`StereoTexWidth`×`StereoTexHeight` = 8×1,
  `D3DFMT_A32B32G32R32F` / `DXGI_FORMAT_R32G32B32A32_FLOAT`, `NVSTEREO_IMAGE_SIGNATURE` =
  `0x4433564E`) read from the header as vendored unmodified in **3Dmigoto** (bo3b et al., credited
  above as a tool); and **`nvapi_interface.h`** in NVIDIA's public NVAPI repository, used as the
  first-party function-name-to-dispatch-ID mapping that confirms which NVAPI stereo entry point a
  binary is (or is not) calling. Read as documentation; no code taken.
  [using nvstereo.h](https://archive.docs.nvidia.com/gameworks/content/technologies/desktop/nv3dva_using_nvstereoh.htm) ·
  [nvstereo.h via 3Dmigoto](https://github.com/bo3b/3Dmigoto/blob/master/nvstereo.h) ·
  [nvapi_interface.h](https://github.com/NVIDIA/nvapi/blob/main/nvapi_interface.h)
- **Epic Games**, additionally — Epic's own **UDK documentation** for *"Unreal Engine 3 and NVIDIA 3D
  Vision Direct"* (the `AllowNvidiaStereo3d` ini key, the fullscreen-only and not-in-editor
  restrictions) and for the **UE2 Runtime**, whose `RuntimeHeaders` page records that the engine's
  C++ native headers were NDA-gated and that licensees used them to build *"360 degree rendering
  drivers for VR systems"* — cited here as first-party corroboration that the pluggable render-device
  interface is the sanctioned extension point on that generation. Both pages return HTTP 403 to
  automated fetch and are cited from their titles and from well-attested search-indexed content;
  treated as `[reported]` accordingly.
  <https://docs.unrealengine.com/udk/Three/ThreeDVision.html> ·
  <https://docs.unrealengine.com/udk/Two/RuntimeHeaders.html>
- **DHR** and **Rubini**, with the wider **3D-fix archive community** — the published stereoscopic-3D
  fixes for Mad Max (DHR's 2015 3Dmigoto fix; Rubini's 2024 geo-11 revision built on it, targeting the
  final shipped game version). Used here only as **reports on engine behaviour** — which passes break
  under stereo in that renderer, and the fact that a fix's own files, not its announcement, are where
  register-level detail lives. **Read online only, never installed or copied.**
- **The `broadside` project wiki** — the public write-up of the AnvilNext `.forge` container format,
  used here for the `scimitar` header identifier and the header/resource-index/resource-data
  structure, and for its explicit statement that public analysis stops at the resource boundary.
- **Turfster** — **Elika**, the Prince of Persia (2008) `.forge` extractor/replacer, and the more
  generic `.forge` tooling; and the maintainers of **AnvilToolkit** for later titles in the family.
  Cited for their published capability lists, which is what establishes that public tooling reaches
  assets but not behaviour graphs. Read online; not used to extract anything for this library.
- **Jill (`scrunguscrungus`)** — **Astralathe**, an all-in-one mod loader, debugging tool, API extender
  and patcher for the modern Psychonauts release (in beta, hosted on GitLab), together with the
  ecosystem built on it (PsychoRando; the Psychonauts Archipelago integration). Cited here as the
  worked example of *check whether the community already built it — and whether it collides with your
  proxy*. Not installed by this account at the time of writing.
- **RayCarrot** — **PsychonautsStudio** (**MIT**), a file-format tool covering all versions of the
  game including console builds, with serialization logging. Cited as public tooling context.

- **The Unreal Wiki**, as preserved by **Unreal Archive** — the *Unreal Unit* page: the 16 UU per foot
  / ≈52.5 UU per metre convention, its explicit statement that there is no fixed relation to real-world
  units, and the per-game player-collision table (Unreal / Unreal Tournament: 78 UU tall). Read as
  public documentation; cited to bound the Unreal Gold project's IPD-default hypothesis.
  <https://unrealarchive.org/wikis/unreal-wiki/Unreal_Unit.html>
- **OldUnreal**, additionally — the public 227 UnrealScript source (`Unreal-PubSrc`), read online as
  documentation for the script-event camera finding on the UE1–3 family page; and **`[]KAOS[]Casey`**
  and **`han`** on the OldUnreal forums, for the published native-package build-and-bind procedure
  summarised there. **BeyondUnreal wiki** (community-preserved) for its "customising the player view"
  page. All surfaced by the `unreal-gold-vr` project's own `/gr` research pass; no code taken.
  <https://github.com/OldUnreal/Unreal-PubSrc> ·
  <https://www.oldunreal.com/phpBB3/viewtopic.php?t=3938> ·
  <https://beyondunrealwiki.github.io/pages/customising-the-player-view.html>

### Our own first-party research

The [read the shipped files](./docs/techniques/README.md#read-the-shipped-files-before-you-attach-anything),
[counting events is not measuring content](./docs/techniques/README.md#counting-events-is-not-measuring-content),
[stereo early-out](./docs/techniques/README.md#stereo-hazard-a-setter-that-early-outs-on-an-unchanged-matrix)
and [composition bugs that masquerade as handedness](./docs/techniques/README.md#composition-bugs-that-masquerade-as-handedness)
sections, and the Avalanche, Dunia, id Tech 5 and UE1–3 family pages' camera-delivery findings, were
generalised out of our own static analysis of legitimately-owned copies of **Mad Max**, **Enslaved**,
**The Evil Within**, **Far Cry 2** and **XIII (2003)**. Full evidence in each project's
`engine-research/` and `modding-notes/` folders:
[`mad-max-vr`](https://github.com/TefMeister/mad-max-vr) ·
[`enslaved-vr`](https://github.com/TefMeister/enslaved-vr) ·
[`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr) ·
[`far-cry-2-vr`](https://github.com/TefMeister/far-cry-2-vr) ·
[`XIII2003-vr`](https://github.com/TefMeister/XIII2003-vr).

The [control rules](./docs/techniques/README.md#controls-a-negative-needs-a-positive-one-a-positive-needs-a-no-op-one),
the [write-combined memory section](./docs/techniques/README.md#never-cpu-scan-mapped-gpu-memory-in-place--it-is-write-combined)
and the [console-automation section](./docs/techniques/README.md#driving-a-game-console-with-synthetic-keys-scancodes-layouts-and-dead-keys)
were generalised out of the DOOM (2016) work, including several of that project's own withdrawn
claims — kept and cited because the corrections are the transferable part.

The [third-party stereo fix as intelligence](./docs/techniques/README.md#a-third-party-stereo-fix-is-free-intelligence-about-the-engine--read-it-dont-install-it),
[proxy export completeness](./docs/techniques/README.md#a-proxy-dll-must-export-everything-the-target-actually-imports)
and [the instrument can be the bug](./docs/techniques/README.md#the-instrument-can-be-the-bug)
sections were generalised out of the 2026-08-25 static-recon and first-injection work on
legitimately-owned copies of **Alice: Madness Returns**, **Alan Wake**, **Prince of Persia (2008)**
and **Burnout Paradise**:
[`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr) ·
[`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr) ·
[`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr) ·
[`burnout-paradise-vr`](https://github.com/TefMeister/burnout-paradise-vr). The
signal-must-separate-the-states guard came from
[`arcade-controls-re2-vr`](https://github.com/TefMeister/arcade-controls-re2-vr).

The [call-argument switch shape](./docs/techniques/README.md#the-switch-you-cannot-find-may-be-an-argument-not-a-global)
and the [repeated-launch ASLR sampling trap](./docs/techniques/README.md#a-repeated-launch-is-not-an-aslr-test)
were likewise generalised out of the DOOM (2016) work — both of them out of *withdrawn* claims, which
is again where the transferable content was:
[`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr). The underlying cvar-dump read that
settled the first came via that project's `/gr` research pass.

The [id Tech 6 case study](./docs/case-studies/id-tech-6-dormant-stereo.md) is our own static
analysis of a legitimately-owned copy, not a summary of someone else's work. The full evidence and
working notes are published openly at
[`doom-2016-vr/engine-research/`](https://github.com/TefMeister/doom-2016-vr/tree/main/engine-research),
[`doom-2016-vr/dev-archive/`](https://github.com/TefMeister/doom-2016-vr/tree/main/dev-archive), and
[`doom-2016-vr/modding-notes/`](https://github.com/TefMeister/doom-2016-vr/tree/main/modding-notes).

The [RenderWare packed-binary case study](./docs/case-studies/packed-binary-live-memory-scan.md) is
likewise our own static-and-live analysis of a legitimately-owned copy of Manhunt (2003). Full
evidence at
[`manhunt-2003-vr/engine-research/`](https://github.com/TefMeister/manhunt-2003-vr/tree/main/engine-research),
[`manhunt-2003-vr/dev-archive/`](https://github.com/TefMeister/manhunt-2003-vr/tree/main/dev-archive), and
[`manhunt-2003-vr/external-research/`](https://github.com/TefMeister/manhunt-2003-vr/tree/main/external-research).

The [harness tick-sites section](./docs/techniques/README.md#driving-a-live-game-from-a-hook) and
the UE1–3 family page's automation findings come from our own live work on a legitimately-owned
copy of XIII (2003). Full evidence — including the **disproved** render-path diagnosis, kept on
purpose so nobody re-walks it — at
[`XIII2003-vr/engine-research/`](https://github.com/TefMeister/XIII2003-vr/tree/main/engine-research),
[`XIII2003-vr/dev-archive/`](https://github.com/TefMeister/XIII2003-vr/tree/main/dev-archive), and
[`XIII2003-vr/modding-notes/`](https://github.com/TefMeister/XIII2003-vr/tree/main/modding-notes).

The [void-behind-the-player](./docs/techniques/README.md#the-void-behind-the-player) and
[camera-matrix](./docs/techniques/README.md#finding-the-camera-matrix-the-engine-actually-reads)
sections, and the [Double Fine engine page](./docs/engines/double-fine-psychonauts.md), are our own
live analysis of a legitimately-owned copy of Psychonauts (2005), including the measured void
percentages and the three failed hypotheses that preceded the answer. Evidence at
[`psychonauts-vr/modding-notes/`](https://github.com/TefMeister/psychonauts-vr/tree/main/modding-notes)
and [`psychonauts-vr/dev-archive/`](https://github.com/TefMeister/psychonauts-vr/tree/main/dev-archive).

The [HMD-anchored body float](./docs/techniques/README.md#vr-body-height-the-hmd-anchored-float) and
the list of animation-layer approaches that **failed** to fix it come from our own work on a
legitimately-owned copy of Resident Evil 2 Remake (2019). Evidence at
[`visceral-re2-vr/modding-notes/`](https://github.com/TefMeister/visceral-re2-vr/tree/main/modding-notes).

The RE Engine family page's [argument-encoding silent-zero
finding](./docs/engines/re-engine.md#scalar-floats-passed-to-invoke-from-the-native-c-plugin-sdk-can-land-as-zero),
the mirror render-layer control surface, and the
[hook-to-acquire-a-handle](./docs/techniques/README.md#hook-to-acquire-a-handle-the-api-will-not-give-you)
pattern are our own measurements on a legitimately-owned copy of Resident Evil Village (2021).
Evidence at
[`re-village-scope-vr/modding-notes/`](https://github.com/TefMeister/re-village-scope-vr/tree/main/modding-notes)
and [`re-village-scope-vr/external-research/`](https://github.com/TefMeister/re-village-scope-vr/tree/main/external-research).
The same project's [scoped-optics finding](./docs/techniques/README.md#a-flat-games-scope-is-a-fullscreen-fov-zoom-and-vr-cannot-use-it)
— that a flat sniper scope is a fullscreen FOV zoom plus a GUI mask, with the measured 63° → 24.37°
ramp — is ours as well.

The [resource-identity](./docs/techniques/README.md#identify-a-resource-by-how-it-is-used-not-by-its-creation-descriptor)
and [deferred-context](./docs/techniques/README.md#deferred-context-renderers-finding-the-world-and-patching-it-once-per-eye)
sections and the [id Tech 5 family page](./docs/engines/id-tech-5.md) come from our own live analysis
of a legitimately-owned copy of The Evil Within (2014), including the decoy-buffer rounds that worked
perfectly on the wrong data — kept because that is the transferable part. Evidence at
[`the-evil-within-vr/engine-research/`](https://github.com/TefMeister/the-evil-within-vr/tree/main/engine-research)
and [`the-evil-within-vr/dev-archive/`](https://github.com/TefMeister/the-evil-within-vr/tree/main/dev-archive).

The [whole-frame capture route](./docs/techniques/README.md#capturing-the-finished-frame-the-whole-frame-route-to-a-headset)
and the [focus-gated main loop](./docs/techniques/README.md#an-old-main-loop-may-stop-rendering-the-moment-it-loses-focus)
are our own hardware-verified work on a legitimately-owned copy of XIII (2003), profiling numbers
included. The
[registry-driven-setting](./docs/techniques/README.md#the-setting-you-want-to-change-may-be-data-not-code),
[one-launch](./docs/techniques/README.md#make-one-launch-answer-many-questions) and
[remove-your-own-code](./docs/techniques/README.md#remove-your-own-code-before-accepting-the-blame--then-fix-the-producer)
sections, and the RenderWare family page's injection-route notes, come from the Manhunt (2003) work
credited above.
The [debug-the-right-value](./docs/techniques/README.md#prove-the-value-you-are-debugging-is-the-one-the-feature-reads)
section is from the Resident Evil 2 Remake work credited above.

The [mutation-checked verification](./docs/techniques/README.md#prove-the-test-can-fail-mutation-check-a-numerical-verification-before-trusting-it)
section and the UE1 camera-delivery findings on the UE1–3 family page are our own work on a
legitimately-owned copy of Unreal Gold under OldUnreal's 227k patch —
[`unreal-gold-vr`](https://github.com/TefMeister/unreal-gold-vr). The
[byte-identity read-only-tree rule](./docs/techniques/README.md#when-byte-identity-is-the-evidence-the-tree-is-read-only)
is from the XIII (2003) work credited above; the
[eye-height](./docs/techniques/README.md#measuring-eye-height-for-a-first-person-conversion-camera-minus-player-is-a-camera-height)
and unbound-key findings are from the Psychonauts work credited above; and the stage-qualified
register correction and the per-licensee NVIDIA-branch finding on the UE1–3 page are from the Enslaved
work credited above, each with full evidence in that project's `modding-notes/` and `engine-research/`.

The [shader-compile-time table](./docs/techniques/README.md#when-a-game-compiles-its-shaders-decides-how-you-read-its-constant-map)
generalises across `enslaved-vr`, `alice-madness-returns-vr`, `mad-max-vr` and `alan-wake-vr`'s own
first-party research, all credited above; its runtime-compilation row draws on public Steam Community
troubleshooting threads that document Alan Wake's "could not process hlsl shader" launch failure,
read as evidence of runtime shader compilation only, no code or files taken.

The [OpenXR per-view pose section](./docs/techniques/README.md#openxr-carries-a-pose-per-view-where-openvr-collapses-to-one)
credits **LukeRoss00**, who filed [OpenVR issue #1253](https://github.com/ValveSoftware/openvr/issues/1253)
(still open, no Valve response as of 2026-09-02) describing exactly the per-eye-pose defect a
same-frame AER submission runs into; and **the Khronos Group**, whose published
[OpenXR SDK](https://github.com/KhronosGroup/OpenXR-SDK) header (`openxr.h`) was read online to
verify that `XrCompositionLayerProjection` carries an array of per-view poses, closing the question
as a specification fact rather than an inference. Generalised out of `far-cry-2-vr` and `XIII2003-vr`'s
own first-party research, both credited above.

The [encrypted-`.text` entropy signature](./docs/techniques/README.md#packedself-protecting-binaries)
and its [Automatic-vs-Direct caveat](./docs/techniques/README.md#-the-diagnostic-that-matters-for-recon-automatic-vs-direct)
are our own first-party static work on a legitimately-owned copy of Alice: Madness Returns (2011),
credited above. The
[compressor/type-hash section](./docs/techniques/README.md#the-executable-can-name-its-own-compressed-formats-and-type-hashes)
and its positive-control addition to the [controls
rules](./docs/techniques/README.md#controls-a-negative-needs-a-positive-one-a-positive-needs-a-no-op-one)
are our own first-party static work on a legitimately-owned copy of Prince of Persia (2008), credited
above under **LZO**; full evidence in
[`prince-of-persia-2008-vr/dev-archive/`](https://github.com/TefMeister/prince-of-persia-2008-vr/tree/main/dev-archive)
(`tools/forge/FORMAT.md`). The [GitLab REST API
method](./docs/techniques/README.md#tool-defaults-that-fabricate-false-negatives) came from the
`psychonauts-vr` project's own research toolbox, applied by a `/gr` sweep to read Astralathe's
GitLab-hosted source.

Like everything else we write, these are CC-BY-4.0 — take them and build on them, just say where
they came from.

---

*This library is non-commercial and educational. All trademarks and copyrights belong to their
respective owners.*
