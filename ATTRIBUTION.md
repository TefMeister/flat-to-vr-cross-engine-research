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
- **porlock2** — [REFramework PR #1822](https://github.com/praydog/REFramework/pull/1822) (merged
  2026-09-05), fixing a VR multipass startup crash caused by a March 2026 game patch re-laying-out
  `via.render.Texture`, with the new offsets measured rather than estimated. Cited as the worked
  public example in
  [techniques → the version that moves is usually the game's](./docs/techniques/README.md#-the-version-that-moves-is-usually-the-games--and-a-moved-struct-field-crashes-with-your-framework-nowhere-in-the-stack)
  and on the [RE Engine family page](./docs/engines/re-engine.md).
- **prideslayer** and contributors — **VRIK Player Avatar** (Skyrim VR). Cited only to draw the
  distinction between the well-known VR floor-calibration/height-offset problem it addresses and the
  pose-dependent, animation-driven body float documented in
  [techniques](./docs/techniques/README.md#vr-body-height-the-hmd-anchored-float). No code or
  technique reused.
  - <https://www.nexusmods.com/skyrimspecialedition/mods/23416>

## Tools, drivers & communities

- **Fire-Head** — **MHNoDRM**, the community write-up documenting Manhunt (2003)'s 16 SecuROM-remnant
  IAT-hook addresses and their fake-return-value mechanism. Technique studied and independently
  verified against our own live memory dump; no code reused. Also **MHWSF**, the same author's
  public widescreen fix for the game, whose published camera/screen globals (the view-window pair
  that RenderWare uses in place of a projection matrix, and the `RwCamera` pointer beside it) are
  cited as a report about a binary we own — read online, verified independently in our own process,
  no code reused.
  <https://github.com/Fire-Head/MHNoDRM>
- **SirKandela** (Chaos LTD) and **Rylie Pavlik** — the 2023 Khronos-forum thread reporting an
  OpenXR desktop runtime holding both projection views on the HMD pose regardless of the poses
  submitted, and the reply explaining why a runtime that truly ignored the pose could not reproject
  correctly. Read online as a report on runtime behaviour; nothing taken.
  <https://community.khronos.org/t/oculus-runtime-ignores-projection-layer-views-pose/110078>
- **eqzitara** — the HelixMod 3D Vision fix published for Enslaved: Odyssey to the West (Premium
  Edition), 2013. Cited for its public description of which passes it had to correct and its
  motion-blur requirement, which independently corroborated our own static prediction on the same
  binary. Closed source; read online, never installed, no code taken.
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
  emulation, and the driver behind this library's
  [measured virtual-pad input route](./docs/techniques/README.md#known-input-routes-by-engine-family):
  a virtual Xbox 360 pad it creates is bound by an XInput game as a genuine controller, focus-
  independently and with nothing injected. <https://github.com/nefarius/ViGEmBus>
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

- **atom0s** — **Steamless**, the open-source SteamStub (Steam DRM wrapper) unpacker, explicitly for
  software you own. Read online for its supported-variant list and stated purpose; nothing copied.
  Its existence is what turns an encrypted `.text` from a runtime-dump problem into a static one.
  <https://github.com/atom0s/Steamless>
- **GHFear** — **Steamstub-v3-Unpacker**, a second open-source SteamStub v3 unpacker; read online
  only. <https://github.com/GHFear/Steamstub-v3-Unpacker>
- **Adam Hlt** — "Cube World Reversing — Unpack the game", the write-up that describes the `.bind`
  section, the entry-point redirection into it and `.text` being encrypted at rest.
  <https://adamhlt.com/cube-world-reversing-unpack-the-game/>
- **elishacloud** — **dxwrapper** / **DirectX-Wrappers**, whose `D3d9to9Ex` option and its
  maintainer's discussion of the `D3DPOOL_MANAGED → DEFAULT + DYNAMIC` rewrite (7 of 8 games working,
  and the four named 9Ex limits) are cited as prior art for the D3D9Ex upgrade. **Kaldaien**'s
  **Special K** is credited there as the origin of that strategy. Read online; no code reused.
  <https://github.com/elishacloud/dxwrapper/discussions/105> · <https://github.com/elishacloud/DirectX-Wrappers>
- **Pauldusler** — **3D Fix Manager** and its page's driver-support history (425.31 / 452.06, the
  October 2020 DX11 removal, DX9 compatibility, Discover mode), cited for the 3D Vision status note.
  <https://helixmod.blogspot.com/2017/05/3d-fix-manager.html>
- **davegl1234** — named by Helix Mod as the developer of the **geo-11** replacement stereo driver
  (June 2022); credited alongside **ThreeDeeJay**, who hosts its releases.
  <https://helixmod.blogspot.com/2022/06/announcing-new-geo-11-3d-driver.html>
- **Jim2point0** — the FRAMED-hosted Alan Wake cheat table whose FOV and time-scale byte patterns
  (read online as XML) ported exactly to our build and led to the camera global. Patterns re-derived
  in our own binary; nothing copied. <https://framedsc.com/GameGuides/Alan_Wake.htm>
- **Neovad** — the Helix Mod Alan Wake 3D Vision fix, whose comment thread documents v1.06's
  FOV-dependent shadows. <https://helixmod.blogspot.com/2014/08/alan-wake.html>
- **gho** — **DxWnd** (windowed-mode wrapper for fullscreen games, SourceForge). The public
  diagnosis, in a 2014 thread on D3D9 device-`Reset` trouble, that `BeginStateBlock` restores the
  device's COM method pointers and thereby invalidates in-place hook patching — and that hooking that
  method is the fix. The primary witness behind our
  [state-block section](./docs/techniques/README.md#recording-a-state-block-rewrites-the-devices-method-table--and-your-in-place-vtable-patch-with-it).
  <https://sourceforge.net/p/dxwnd/discussion/general/thread/9b1c8171/> · project:
  <https://sourceforge.net/projects/dxwnd/>
- **Paul Roussin** — the corroborating D3D8-era statement, on Microsoft's now-retired DirectX graphics
  newsgroup, that `BeginStateBlock` resets the device table and a vtable hook must re-apply its
  addresses afterwards. Survives only on a third-party Usenet archive, whose displayed date we could
  not corroborate; cited as an archived public post, not a vendor source.
  <https://microsoft.public.win32.programmer.directx.graphics.narkive.com/PbJcO31s/hooking-d3device8-by-replacing-the-vtable-fails-info-needed>
- **crosire** and the ReShade contributors — additionally credited for **ReShade commit `74347b91d`**
  (2019-12-19, shipped in 4.5.2), *"Fix hooking in Alan Wake"*, whose diff comment records that freeing
  the module reference taken for export hooks is what makes that game work. Prior art for our
  [free-the-real-DLL section](./docs/techniques/README.md#and-it-must-free-the-real-dll-on-detach-or-a-reload-walks-straight-past-it);
  read online only, no code taken.
  <https://github.com/crosire/reshade/commit/74347b91d7729a6da93040298c6587bb3b786da4>
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
- **The Vulkan specification** (Khronos) — the Memory Allocation chapter, cited for what
  `VK_MEMORY_PROPERTY_HOST_COHERENT_BIT` means for the host cache-management commands, which is what
  settles [a legal-but-unnecessary call is not evidence of a mechanism](./docs/techniques/README.md#-the-inverse-a-legal-but-unnecessary-call-is-not-evidence-of-a-mechanism).
  <https://docs.vulkan.org/spec/latest/chapters/memory.html>
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
- **Hajime Uchimura** (Polyphony Digital) — *"HDR theory and practice"*, CEDEC 2017: the **GT
  tonemap** (a three-section curve: toe, exactly linear middle, shoulder), whose published default
  parameters are the fingerprint by which RE Engine's `via.render.ToneMapping` was identified on the
  [RE Engine page](./docs/engines/re-engine.md#the-games-tone-curve-is-a-published-one-and-its-parameters-are-readable).
  Read online as documentation; our implementation was written from the published formula in our own
  words and code. [slides](https://www.slideshare.net/nikuque/hdr-theory-and-practicce-jp) ·
  [Polyphony's HDR/WCG paper on the same curve](http://cdn2.gran-turismo.com/data/www/pdi_publications/PracticalHDRandWCGinGTS_20181222.pdf)
- **Microsoft** — the D3D11 reference for
  [`VSSetConstantBuffers`](https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11devicecontext-vssetconstantbuffers),
  which documents that a bound constant buffer may exceed what a shader addresses — the fact behind
  the "a live size no shader declares" note in the
  [reflection → disassembly section](./docs/techniques/README.md#reflection-gets-you-to-n-unnamed-slots-disassembly-names-them-by-use);
  and `fxc`, the shader compiler whose `-dumpbin` disassembly that section relies on.
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
  proxy*. Not installed by this account at the time of writing. Additionally credited, from the
  2026-09-03 sweep, as the **signature source** worked example: Astralathe publishes byte signatures
  for engine functions, and scanning those against our own legitimately-owned executable corroborated
  several independently-found addresses and recovered the engine's own names for them — a method that
  takes a pattern, not code. The same author's **PsychoPortal** level-format work is the source of the
  `VisibilityTree`/PVS structure described on the engine page. Read online only; no code taken, and
  the GPLv3 terms on their repository govern their code, which we do not use.
  <https://gitlab.com/scrunguscrungus/astralathe> · <https://gitlab.com/scrunguscrungus/psychoportal>
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

- **Microsoft** — Win32 API documentation, cited directly where a rule of the platform decides a
  technique: the `LoadLibraryA` remark that a bare module name resolves to an already-resident module
  of the same base name; `CreateProcess`'s environment-inheritance behaviour; and the `DllMain`
  remarks that an entry point must not call `FreeLibrary` during process termination, together with
  the rule that distinguishes the two detach cases by the third parameter.
  <https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibrarya> ·
  <https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessw> ·
  <https://learn.microsoft.com/en-us/windows/win32/dlls/dllmain> ·
  <https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-best-practices>
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

The [reflection → disassembly](./docs/techniques/README.md#reflection-gets-you-to-n-unnamed-slots-disassembly-names-them-by-use)
and [per-pass filter](./docs/techniques/README.md#-a-constant-across-every-draw-in-the-frame-filter-excludes-a-per-pass-camera-by-design)
sub-sections, the [D3D9 `Reset` section](./docs/techniques/README.md#a-d3d9-reset-can-disarm-a-device-hook-silently-and-late)
and the [one-file-two-compilers section](./docs/techniques/README.md#test-a-runtime-compiled-shader-without-the-game-one-file-two-compilers)
are our own first-party research on legitimately-owned copies of Mad Max, Enslaved: Odyssey to the
West and Resident Evil Village, generalised on 2026-09-04 out of each project's notes:
[`mad-max-vr`](https://github.com/TefMeister/mad-max-vr) ·
[`enslaved-vr`](https://github.com/TefMeister/enslaved-vr) ·
[`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr). The `dxbc-usage.py` tool
those sections mention is ours, in
[`flat-to-vr-RE-toolkit`](https://github.com/TefMeister/flat-to-vr-RE-toolkit). Where they rest on
someone else's public work — Microsoft's D3D11 documentation and `fxc`, Hajime Uchimura's published
GT tonemap — that work is credited above and was read online only.

The [state-block vtable rewrite](./docs/techniques/README.md#recording-a-state-block-rewrites-the-devices-method-table--and-your-in-place-vtable-patch-with-it),
[free the real DLL on detach](./docs/techniques/README.md#and-it-must-free-the-real-dll-on-detach-or-a-reload-walks-straight-past-it),
[shader assembly text in the binary](./docs/techniques/README.md#a-fourth-case-the-shader-is-assembly-text-and-it-ships-in-the-binary),
[one edit for both pipelines](./docs/techniques/README.md#if-both-pipelines-read-the-same-transform-per-eye-stereo-is-one-edit),
[photo mode as a constants testbed](./docs/techniques/README.md#a-photo-mode-is-also-a-free-testbed-for-the-camera-and-projection-constants),
[synthetic tap length and reading the keymap file](./docs/techniques/README.md#4-a-synthetic-tap-has-a-minimum-length-and-it-is-per-machine),
[configure from a file, not the environment](./docs/techniques/README.md#configure-injected-code-from-a-file-it-reads-itself-not-from-environment-variables)
and [the gated diagnostic](./docs/techniques/README.md#the-diagnostic-that-is-gated-on-the-failure-it-was-written-to-explain)
sections were generalised on 2026-09-04 out of our own work on legitimately-owned copies of
**Enslaved: Odyssey to the West**, **Alan Wake**, **XIII (2003)**, **Mad Max** and **The Evil Within**:
[`enslaved-vr`](https://github.com/TefMeister/enslaved-vr) ·
[`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr) ·
[`XIII2003-vr`](https://github.com/TefMeister/XIII2003-vr) ·
[`mad-max-vr`](https://github.com/TefMeister/mad-max-vr) ·
[`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr). Two of them also record a
**withdrawal** — the "120–240 frames after `Reset`" latency and the "this UE3 build stripped its exec
dispatch" reading, both retracted by the projects themselves on 2026-09-04 — and those corrections are
kept in place here, because knowing why a measurement was wrong transfers further than the measurement
would have. Where these rest on other people's public work — Microsoft's Win32 documentation, gho's
DxWnd diagnosis, Paul Roussin's newsgroup answer and ReShade's Alan Wake fix — that work is credited
above and was read online only; no code was taken from any of it.

The [one-element per-eye edit](./docs/techniques/README.md#-and-then-the-edit-itself-is-one-element-not-a-rebuilt-matrix),
[prove an effect by reversing it](./docs/techniques/README.md#prove-an-effect-by-reversing-it-not-by-scoring-it),
[check the scan's base before its range](./docs/techniques/README.md#when-a-scan-finds-nothing-check-its-base-before-you-widen-its-range),
the measured **virtual-pad input route**, and the program-order correction to our own `dxbc-usage.py`
census were generalised on 2026-09-04 out of legitimately-owned copies of **Mad Max**, **Alice:
Madness Returns** and **DOOM (2016)**:
[`mad-max-vr`](https://github.com/TefMeister/mad-max-vr) ·
[`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr) ·
[`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr). Two of those entries exist because a
project **corrected itself**: an "inconclusive" stereo run that turned out to be gated instrumentation
rather than a negative result, and a scan fix that was real but was not the cause of the symptom it was
made to explain. The virtual-pad route rests on **Nefarius's ViGEmBus**, credited above; no code was
taken from it.

The [vtable-patch lifetime](./docs/techniques/README.md#a-vtable-patch-is-a-lifetime-commitment--restore-it-before-anything-can-unload-you),
[alternate-eye parity](./docs/techniques/README.md#alternate-eye-rendering-latch-the-eye-with-the-frame-or-you-silently-swap-them),
[the absurd-transform coverage test](./docs/techniques/README.md#-the-cheapest-coverage-test-is-an-absurd-transform-and-it-is-a-picture)
and the [condition on the one-element edit](./docs/techniques/README.md#-it-is-a-different-stereo-not-a-shorter-way-to-write-the-same-one--and-one-condition-on-preferring-it)
were generalised on 2026-09-04 out of legitimately-owned copies of **Alan Wake**, **Far Cry 2**, **The
Evil Within** and **Alice: Madness Returns**:
[`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr) ·
[`far-cry-2-vr`](https://github.com/TefMeister/far-cry-2-vr) ·
[`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr) ·
[`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr). Three of those
entries exist because a project **overturned something already written down**: a "confirmed broken,
cause unknown" hook verdict that turned out to be an ordinary dangling pointer, a residual assumed
harmless that was carrying world geometry, and this library's own advice to prefer the one-element
per-eye edit, which a project showed would have desynchronised its two shader stages. Where they rest
on other people's public work — Microsoft's `DllMain` and loader documentation, NVIDIA's published
3D Vision Automatic material, and ReShade's Alan Wake fix — that work is credited above and was read
online only.

The [legal-but-unnecessary call](./docs/techniques/README.md#-the-inverse-a-legal-but-unnecessary-call-is-not-evidence-of-a-mechanism),
[enumerate every CPU write path](./docs/techniques/README.md#enumerate-every-cpu-write-path-to-a-constant-buffer-before-believing-your-coverage)
and [dating a dependency](./docs/techniques/README.md#dating-a-dependency-a-fix-newer-than-your-build-is-not-evidence-that-you-are-affected)
sections were generalised on 2026-09-04 out of legitimately-owned copies of **DOOM (2016)**, **The
Evil Within** and **Resident Evil 2**:
[`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr) ·
[`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr) ·
[`visceral-re2-vr`](https://github.com/TefMeister/visceral-re2-vr). The first rests on the Khronos
Group's published Vulkan specification, credited above and read online. The same day,
[`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr) confirmed the free-the-real-DLL fix live,
which is why that section now carries a `[verified-live]` tag and a named acceptance test rather than
prior art alone.

The [own the getter the solver reads](./docs/techniques/README.md#-when-every-setter-is-a-dead-end-own-the-getter-the-solver-reads),
[search a reflection table's doc comments](./docs/techniques/README.md#-a-reflection-table-often-carries-the-developers-own-doc-comments--search-those-not-the-names),
[`(context, resource)` map pairing](./docs/techniques/README.md#-and-an-in-flight-maps-identity-is-context-resource--never-the-resource-alone),
[the reload race on a foreign vtable slot](./docs/techniques/README.md#-and-the-reload-the-fix-enables-has-a-race-of-its-own-the-slot-you-chain-into-may-not-be-the-engines),
[the convention-blind instrument](./docs/techniques/README.md#an-instrument-that-tests-one-convention-can-only-ever-report-neither-convention),
[the co-occurring log line](./docs/techniques/README.md#a-log-line-that-co-occurs-with-a-failure-is-not-an-explanation-of-it),
[three controls for an exhaustive static negative](./docs/techniques/README.md#1b-the-static-search-version-three-controls-turn-an-exhaustive-negative-into-evidence),
[the sweep that changes nothing](./docs/techniques/README.md#and-the-sweep-that-changes-nothing-is-a-positive-result-about-where-the-cause-is-not)
and [the ModRM hole in a cross-reference scanner](./docs/techniques/README.md#-a-cross-reference-scanner-that-does-not-decode-modrm-is-blind-on-x64--and-every-no-xrefs-result-it-produced-is-suspect)
sections were generalised on 2026-09-05 out of our own work on legitimately-owned copies of
**Resident Evil 2**, **DOOM (2016)**, **The Evil Within**, **Alan Wake**, **Prince of Persia (2008)**
and **Resident Evil Village**:
[`visceral-re2-vr`](https://github.com/TefMeister/visceral-re2-vr) ·
[`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr) ·
[`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr) ·
[`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr) ·
[`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr) ·
[`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr). The cross-reference
section is about a defect in **our own** tool, `static-disasm.py` in
[`flat-to-vr-RE-toolkit`](https://github.com/TefMeister/flat-to-vr-RE-toolkit) — recorded here in full
because a tool that produces confident false negatives has to be documented at least as loudly as one
that works. Four of these entries exist because a project **corrected itself within a day**: a
thread-pool diagnosis that named the wrong subsystem, an instrument that could only ever report
"neither convention", a "setter with a blend time is exactly what we need" reading that turned out to
be a dead end, and an absolute-target design that had to become a relative one.

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

The section's [2026-09-03 addition on runtime
behaviour](./docs/techniques/README.md#-expressible-is-not-honoured--two-public-reports-two-runtimes-opposite-directions)
credits **LukeRoss00** a second time, for his 2020 report on Valve's SteamVR discussion board that
spec-correct per-view poses produced a wrong stereo baseline and a vertical inter-eye offset on that
runtime, together with the workaround he published; and **SirKandela** and **Rylie Pavlik**, credited
above, for the 2023 thread that reports the opposite runtime behaving the opposite way. Both are read
purely as public reports on runtime behaviour. The specification wording quoted alongside them is the
**Khronos Group's** own `XrCompositionLayerProjectionView` reference page in the
[OpenXR registry](https://registry.khronos.org/OpenXR/specs/1.1/man/html/XrCompositionLayerProjectionView.html).

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


The [per-draw stereo coverage
section](./docs/techniques/README.md#per-draw-stereo-reaches-only-the-draws-that-read-the-transform-you-hooked),
the [projection-matrix-free frustum
section](./docs/techniques/README.md#the-engine-may-have-no-projection-matrix-to-patch), the
[public-reimplementation signature
method](./docs/techniques/README.md#a-public-reimplementation-of-your-game-is-a-signature-source-not-just-a-reference),
the [engine's-own-accessor
rule](./docs/techniques/README.md#match-the-engines-own-accessor-not-the-ideal-maths), the
[PVS-versus-frustum diagnostic](./docs/techniques/README.md#the-void-behind-the-player) and the
[post-processing rule](./docs/techniques/README.md#turn-off-the-post-processes-that-re-derive-the-view-before-judging-a-stereo-run)
are our own first-party research on legitimately-owned copies of XIII (2003), Manhunt (2003),
Psychonauts (2005) and Enslaved: Odyssey to the West, generalised out of each project's
`engine-research/` folder and credited to those projects above. Where they build on someone else's
public work — **Jill (`scrunguscrungus`)**'s Astralathe signatures, **Fire-Head**'s MHWSF globals,
**eqzitara**'s 3D Vision fix, and the public documentation of Psychonauts' level format — that work
is credited above and was read online only; every address, count and measurement quoted was
re-derived in our own binaries.

The [storage-class check](./docs/techniques/README.md#determine-the-matrix-storage-class-two-ways-before-writing-any-per-eye-edit)
and its [fused-matrix `p00` rule](./docs/techniques/README.md#on-a-fused-matrix-p00-cannot-be-recovered-under-object-scale--keep-the-cameras-projection),
the [register-displacement trap](./docs/techniques/README.md#the-register-is-not-fixed-a-skinning-palette-displaces-the-camera-constants),
the [shipped-compiler correction](./docs/techniques/README.md#when-a-game-compiles-its-shaders-decides-how-you-read-its-constant-map),
the [wrapper-naming section](./docs/techniques/README.md#name-the-wrapper-first--on-steam-it-is-usually-steams-own-and-then-unpacking-is-a-static-step),
the [shipped-switch-still-dispatches caution](./docs/techniques/README.md#and-check-that-the-shipped-switch-still-dispatches--a-binding-in-a-config-is-a-lead-not-a-feature)
and the measured extensions to the per-draw-coverage and early-out sections are our own first-party
research on legitimately-owned copies of Alan Wake, Alice: Madness Returns, Prince of Persia (2008),
Enslaved: Odyssey to the West and XIII (2003), generalised out of each project's `engine-research/`
folder and credited to those projects above:
[`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr) ·
[`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr) ·
[`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr) ·
[`enslaved-vr`](https://github.com/TefMeister/enslaved-vr) ·
[`XIII2003-vr`](https://github.com/TefMeister/XIII2003-vr). Where they rest on someone else's public
work — Microsoft's D3D9 documentation, **elishacloud**'s dxwrapper discussion, **atom0s**'s
Steamless, **Jim2point0**'s cheat table, NVIDIA's `nvstereo.h` — that work is credited above and was
read online only; every address, count and measurement quoted was re-derived in our own binaries.

Like everything else we write, these are CC-BY-4.0 — take them and build on them, just say where
they came from.

---

*This library is non-commercial and educational. All trademarks and copyrights belong to their
respective owners.*
