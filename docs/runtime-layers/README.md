# Runtime & display layers around a VR mod

A flat→VR mod never talks to a headset directly — it talks to a **runtime API** (OpenVR or
OpenXR), and between that API and the panels sits a stack of swappable plumbing. Several public
projects live *in* that plumbing, and knowing them pays off twice: they can rescue a mod that
misbehaves on a given runtime, and they show how the layers compose (useful when your own mod
must pick an API — see the [engine-agnostic core](../engine-agnostic-core/)). They also cover
the **reverse direction**: taking VR (or stereo-3D) output back to 3D displays. All credited in
[`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).

---

## OpenComposite — run OpenVR apps on OpenXR, no SteamVR

**OpenComposite** (originally *OpenOVR* by ZNix; GPL-3.0; maintained by community forks) is a
drop-in reimplementation of the **OpenVR API that forwards to OpenXR**: replace the game's (or
mod's) `openvr_api.dll` (or redirect it system-wide with the OpenComposite launcher) and an
OpenVR title runs natively on any OpenXR runtime — no SteamVR in the middle. People use it to
cut SteamVR's overhead on Quest/Rift/Pimax-class headsets or to sidestep SteamVR issues.

**Why it matters to mod authors:** if your mod targets **OpenVR** (often the pragmatic choice —
e.g. it was long the only 32-bit-friendly option, because SteamVR shipped no 32-bit OpenXR runtime until 2.17),
OpenComposite is the escape hatch that can still land it on OpenXR — subject to bitness: the
target **OpenXR runtime** must exist for the game's architecture. ⚠️ **Corrected 2026-10-11:** SteamVR
**2.17 (stable 2026-09-10) ships a 32-bit OpenXR runtime** `[reported 2026-10-11]`. It is registered under
the `WOW6432Node` `Khronos\OpenXR\1\ActiveRuntime` key only once SteamVR's "set as default OpenXR runtime"
prompt is accepted; until then the file (`steamxr_win32.json`) exists but the 32-bit key stays empty
`[measured 2026-10-06]` (alan-wake-vr). A runtime can also be picked per process with `XR_RUNTIME_JSON`.
Not yet run end to end on our machines. Before 2.17, Virtual Desktop's VDXR was the 32-bit option. It's also a
proof-of-concept that the two APIs are close enough to bridge — with known edge cases around
overlays and some input paths.

- [gitlab.com/znixian/OpenOVR](https://gitlab.com/znixian/OpenOVR) (original) ·
  [community fork README](https://github.com/DevOculus-Meta-Quest/OpenComposite/blob/openxr/README.md)

## OpenXR Toolkit — the post-runtime tuning layer

**OpenXR Toolkit** (Matthieu Bucchianeri / mbucchia; MIT) is an OpenXR **API layer** that sits
between any OpenXR app and the runtime, adding upscaling (NIS/FSR), fixed & eye-tracked foveated
rendering, resolution overrides, world-scale adjustment, and image tuning — all without touching
the app. Development has ended (the project is in maintenance), but it remains widely used and
is *the* reference example of what an OpenXR API layer can do — a pattern a mod project can
itself adopt for cross-cutting features.

- [mbucchia.github.io/OpenXR-Toolkit](https://mbucchia.github.io/OpenXR-Toolkit/) ·
  [github.com/mbucchia/OpenXR-Toolkit](https://github.com/mbucchia/OpenXR-Toolkit)

Related, same layer of the stack: **vrperfkit** (fholger; already in the
[landscape](../landscape/)) does upscaling/foveation for OpenVR titles.

## VRto3D — the reverse direction: VR content on 3D displays

**VRto3D** (oneup03; LGPL-3.0) is a **SteamVR driver that pretends to be a headset**: VR games
and VR *mods* render into it, and it outputs side-by-side / top-and-bottom / frame-packed /
interlaced stereo for 3D TVs, 3D monitors, and simulated-reality displays. It doesn't make flat
games 3D — it gives everything that already renders VR a way onto glasses-based 3D hardware.

**Why it's in this library:** it closes a loop with the flat→VR families. A game VR-modded with
UEVR/UUVR/an engine adapter can, through VRto3D, be played as high-quality stereo 3D on a
3D display — the modern replacement for the dead 3D Vision ecosystem, and a second audience for
any VR mod you build. The Helix community documents this pairing.

- [github.com/oneup03/VRto3D](https://github.com/oneup03/VRto3D) ·
  [project site](https://oneup03.github.io/VRto3D/) ·
  [Helix Mod write-up](https://helixmod.blogspot.com/2024/07/vrto3d-driver.html)

## Depth3D / SuperDepth3D — z-buffer stereo as a ReShade shader

**SuperDepth3D** (BlueSkyDefender's **Depth3D** project; no repo-level license — see the shader
file headers for the author's terms) generates stereoscopic 3D from the **depth buffer** as a
post-process shader running under **ReShade** — the same z-buffer-reconstruction idea as vorpX's
Z-Buffer mode ([generic drivers](../generic-drivers/)), but free, engine-agnostic wherever
ReShade can access depth, and with many output formats (SbS, TaB, interlaced, anaglyph…). A
companion (`Overwatch.fxh`) auto-configures depth settings for many games. Combined with a
VR-headset ReShade path or a 3D display, it's the lowest-effort "some depth, right now" option —
with z-buffer 3D's usual limits (no true occlusion parallax, HUD/effect artifacts).

- [github.com/BlueSkyDefender/Depth3D](https://github.com/BlueSkyDefender/Depth3D) ·
  [project page](https://blueskydefender.github.io/Depth3D/) ·
  [ReShade](https://reshade.me/)

## How the layers stack

```
game ── engine/injector mod (UEVR, UUVR, adapter…) ── OpenVR ──┬── SteamVR ──► headset
                                                               └── OpenComposite ──► OpenXR runtime ──► headset
game ── OpenXR mod ── [OpenXR Toolkit layer] ── OpenXR runtime ──► headset
VR title or VR mod ── SteamVR + VRto3D driver ──► 3D display (SbS/TaB/interlaced)
flat game ── ReShade + SuperDepth3D ──► 3D display (z-buffer stereo)
```

## Headset-free testing: OpenXR-Simulator and its forks

A mod whose output goes through **OpenXR** can be tested with no headset by pointing it at a desktop
OpenXR runtime that shows both eyes in a window.

- **elliotttate's OpenXR-Simulator** (MIT; the original, created 2025-08-09): a small runtime rendering
  side-by-side stereo in a window, with mouse-and-keyboard pose control and D3D11, D3D12 and OpenGL backends;
  tested with Unity, Unreal through UEVR, and the Steam overlay `[reported, from its README]`.
  <https://github.com/elliotttate/OpenXR-Simulator>. fholger's copy (<https://github.com/fholger/OpenXR-Simulator>,
  2026-03-31) is a fork of it, as GitHub's own fork record shows `[measured 2026-11-01]`; this page named fholger as
  the original until 2026-11-01.
- **Forks of elliotttate's original** (webhead2oo9's, and fholger's) add, by their READMEs `[reported
  2026-09-18]`: a **Vulkan** backend; **headset profiles** with measured per-eye FOV, panel resolution
  and IPD for ten headsets, so a projection bug that only shows at one FOV becomes reproducible at a
  desk; frame timing measured from `xrEndFrame` submissions rather than window repaints (p50/p95 on a
  key); a **32-bit** build; activation per process through `XR_RUNTIME_JSON` so the machine's real
  runtime is untouched; and GDI presentation on the D3D12 path to avoid fighting the Steam overlay.
  <https://github.com/webhead2oo9/OpenXR-Simulator>
- **An MCP server** in that fork gives an agent per-eye screenshots, frame diagnostics and a check for
  flicker in quad layers separately from world motion — i.e. the session's own eyes on a VR frame.
  ⭐ An `openxr-simulator` MCP server is **registered in the home PC's Claude Code session** as of
  2026-09-23 `[measured 2026-09-23: its tools are listed in-session; which fork it wraps not checked]`.
- **The probe pattern** from the same account (written for BetterVR): a standalone program that
  replays one mod's exact OpenXR call sequence and exits 0 if the runtime can serve it — turning a class
  of headset-only failures into a desk check.

- **Test lens-centre code on an asymmetric profile** (added 2026-10-26). A real headset's lenses are off-centre
  per eye; a symmetric simulator lens has its centre at zero, so a wrong sign or a lens shift that also moves the
  eye cannot show. Manhunt's per-eye offset sign was wrong and passed every simulator run, then failed on the first
  Quest 3 wear (see the [RenderWare page](../engines/renderware.md)). webhead2oo9's fork carries measured per-eye
  angles for Quest 2, Quest 3, Quest Pro, Index and Vive Pro 2 (from the HMD Geometry Database); in its current
  source the Quest 3 asymmetric profile is the default, a saved setting can turn it back to symmetric, and the MCP
  server's `set_headset_profile` picks one `[inferred-static 2026-10-26, read online in the fork's source]`. Check
  which one your runs actually used before trusting a lens-centre result.
- **Clicking a VR menu from a screenshot** (elliotttate's fork, 2026-10-24, not yet in a release): the MCP server
  can now hover and click with a simulated controller at a pixel of a captured eye image, aiming from the eye or
  from the hand at a known quad or a scene rectangle; an optional per-application ray correction undoes a game's own
  aim-ray tweak (FlatOut 4 adds 3 cm and 5° to its pointer, and an uncorrected aim missed a small button). Results
  mean "input delivered"; the game's own highlight or menu change is the proof it landed `[reported 2026-10-26, from
  its MCP README]`. <https://github.com/elliotttate/OpenXR-Simulator> This is the piece a headset-free menu driver
  for in-world VR menus has been missing.
- **Three traps in elliotttate's simulator that a game hits without warning** (added 2026-11-11, from
  [`silent-hill-2-remake-vr`](https://github.com/TefMeister/silent-hill-2-remake-vr) via our `/gr` lane, and
  [`the-darkness-vr`](https://github.com/TefMeister/the-darkness-vr)):
  - **A 4096-wide picture cap.** v1.7.1 (2026-10-28) and older advertise and enforce at most 4096 for every swapchain,
    whatever the graphics API `[measured 2026-11-11, from the commit message]`. A game that asks for more fails to start
    VR, and can carry on flat with no error: Silent Hill 2 under UEVR at the headset's own 5376 width did exactly that
    `[verified-live 2026-11-10, n=1, SH2 board]`. Commit 692a66d on `master` (2026-10-29) raises it to 16384 for
    Direct3D 11 and 12 only (system properties, view configuration and the `xrCreateSwapchain` check alike); OpenGL,
    Vulkan and mixed bindings keep 4096, and no release carries it yet `[measured 2026-11-11, releases list]`. Until one
    does: measure below 4096 per eye and scale the load with the engine's own resolution setting.
    <https://github.com/elliotttate/OpenXR-Simulator/commit/692a66d>
    **Worked case of that workaround:** SH2 ran the simulator at 2048 × 2194 per eye (58 % of a Quest 3's pixels over
    Virtual Desktop) and raised `r.ScreenPercentage` to 131 % to match the headset's pixel load; the estimate read
    32.5 ms against the headset's own 33.5 ms `[measured 2026-11-10, n=1 each]`, close enough to rank settings at a
    desk ([SH2 note](https://github.com/TefMeister/silent-hill-2-remake-vr/blob/main/modding-notes/2026-11-10-optimiser-step-3-the-upscaler-was-off.md)).
    The same note found the game had silently rewritten its own settings file and turned its upscaler off, so the
    slow headset baseline had been taken with every pixel at full size: **check the game's upscaler and render scale
    before taking any VR performance baseline** `[measured 2026-11-10]`.
  - **An abort at quit.** Since v1.6.0 (2026-10-26) the simulator's preview window runs on its own thread, which is
    joined only inside `xrDestroyInstance`; a process that exits without destroying the instance destroys a still
    running thread object, which is a defined `abort()` in C++ (fast-fail code 7, inside the simulator's own dll since
    it links its C runtime statically) `[inferred-static 2026-11-11, read online in its source, via our /gr lane]`.
    `master` (2026-11-08) is unchanged on this. UEVR destroys the instance only from a path no quit-time caller was
    found for, so every UEVR quit on v1.6.0+ may abort `[hypothesis]`; webhead2oo9's fork (v1.5.1) has no preview
    thread and should not `[hypothesis, untested]`. A crash at quit under the simulator is therefore not evidence about
    the mod. Detail:
    [SH2 topic](https://github.com/TefMeister/silent-hill-2-remake-vr/blob/main/external-research/topics/2026-11-11-the-quit-abort-is-the-simulators-preview-thread.md).
  - **"The game ignores a polite close" may be the wrong window.** Under the simulator the game's process owns two
    top-level windows, and the simulator's preview comes first in the window list; a close message to it closes only
    the preview and the game plays on. Sent to the game's own window (match its title), The Darkness closed in 0.7 s
    with its own quit in the log `[verified-live 2026-11-10, n=3]`
    ([the-darkness-vr note](https://github.com/TefMeister/the-darkness-vr/blob/main/modding-notes/2026-11-10j-the-polite-close-was-asked-of-the-wrong-window.md)).
    A close script that picks "the process's first window" is likely to pick the preview on any game `[hypothesis]`.

**The limit:** these are OpenXR runtimes. They help only a mod whose VR output goes through OpenXR, not
one that draws stereo into the game's own swap chain. phunkaeg's playbook adds the matching warning
(chapter 09): *a substitute runtime that enforces less than production passes things the headset then
fails*, so a simulator pass is not headset acceptance.

**Another headset-free runtime: chasmlol's `xrsim`** (added 2026-11-07) `[reported 2026-11-02, from its docs]`. Part of
the Chasms VR Framework (<https://github.com/chasmlol/chasms-vr-framework>, `docs/XRSIM.md`): an OpenXR runtime with
no hardware, chosen per process through `XR_RUNTIME_JSON` so the system runtime is left alone, with a scripted head and
both controllers, and both eyes saved per frame. Its `--sim all` mode runs a scripted player through every gesture and
checks that the game reacted: a model for turning a headset-free runtime into a regression suite. Whether it runs
under an RE Engine game has not been tried `[hypothesis]`.

## A game's own old C++ runtime can kill any runtime loaded into it

Generalised out of [`heavy-rain-vr`](https://github.com/TefMeister/heavy-rain-vr) (`dev-archive/recon/2026-10-12-old-cpp-runtime-closes-the-game/`).

Many games ship `msvcp140.dll` and `vcruntime140.dll` in their own folder. Being found first, **that copy is what
every DLL in the game's process gets**, including an OpenXR runtime (or the simulator) that a mod loads in process.
Binaries built with Visual Studio 2022 17.10 or later need runtime 14.40 or later: they build their `std::mutex` in
place, and an older runtime crashes at the first lock `[reported]`. The tell recorded with the evidence: the PE
imports `_Mtx_lock` but not `_Mtx_init_in_situ`.

- **Seen:** Heavy Rain ships 14.16 (2018). The OpenXR Simulator died inside `xrCreateInstance` with the game's copy
  beside a test program, and ran with the Windows copy `[measured 2026-10-12, n=2 each]`. In the game it looked like a
  **silent exit**: no crash event and a 0-byte runtime log, which reads like anything but a runtime-version fault.
- **Cheap checks for any game:** list `msvcp140.dll` in the game folder and read its version; before loading a
  runtime, log the path and version of the `msvcp140.dll` already in the process, and refuse below 14.40 with a log
  line rather than lose the game.
- **Fix under test:** move the game's two files out (archived, not deleted) so Windows' newer copies load. Not yet
  tried live `[hypothesis]`.

## Frame warping as a plug-in: PureDark's AFW

**PureDark's UEVR fork** adds *Alternate Frame Warping*: the game draws fewer pictures and the missing
ones are made by warping pictures that exist — one eye from the other, this frame from the last, or
both. The author reports roughly 60–80% more performance in VR, some artefacts, ~500 MB more VRAM,
D3D12 only `[reported]`. <https://github.com/PureDark/UEVR> (branch `AFW`, releases
`UEVR_AFW_v1.0-beta.1` to `beta.6`, July–August 2026).

What the public files show `[inferred-static 2026-09-21, from our modding lane's read]`:

- A warp needs colour, depth and motion vectors per picture, plus the camera it was taken from and the
  camera it should appear taken from. AFW gets depth and motion vectors by **listening to the game's own
  temporal upscaler call** (DLSS, FSR 2/3 or XeSS), which is why the install notes say to turn DLSS or
  DLAA on first. **A game with no temporal upscaler gives it nothing to listen to.**
- The warp itself ships as a **compiled add-on behind a published header**: three plain D3D12 calls —
  initialise the device, initialise the warp, evaluate a frame — none of which requires Unreal. So in
  principle any D3D12 VR mod that can supply those inputs could call it `[hypothesis; untried, and the
  add-on may check its host]`. **Its licence and terms are not established**: it is PureDark's work, to
  be used only with his permission.
- Known artefact: first-person arms and weapons can show double vision; per-game profile scripts fix
  five listed games. Our wearer saw exactly that, slight, around the hands in the Silent Hill 2 remake,
  and judged it a fair trade for the smoothness `[reported 2026-09-21, n=1 wearer]`.

## Sources

- OpenComposite / OpenOVR — [gitlab.com/znixian/OpenOVR](https://gitlab.com/znixian/OpenOVR) ·
  [DevOculus-Meta-Quest fork](https://github.com/DevOculus-Meta-Quest/OpenComposite/blob/openxr/README.md)
- OpenXR Toolkit — [mbucchia.github.io/OpenXR-Toolkit](https://mbucchia.github.io/OpenXR-Toolkit/)
- VRto3D — [github.com/oneup03/VRto3D](https://github.com/oneup03/VRto3D) ·
  [Helix Mod: VRto3D Driver](https://helixmod.blogspot.com/2024/07/vrto3d-driver.html)
- Depth3D — [github.com/BlueSkyDefender/Depth3D](https://github.com/BlueSkyDefender/Depth3D)
- ReShade (crosire) — [reshade.me](https://reshade.me/)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
