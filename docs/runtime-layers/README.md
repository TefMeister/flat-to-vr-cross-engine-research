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
e.g. it's the only 32-bit-friendly option, since SteamVR ships no 32-bit OpenXR runtime),
OpenComposite is the escape hatch that can still land it on OpenXR — subject to bitness: the
target **OpenXR runtime** must exist for the game's architecture (SteamVR, notably, ships no
32-bit OpenXR runtime). It's also a
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

## Sources

- OpenComposite / OpenOVR — [gitlab.com/znixian/OpenOVR](https://gitlab.com/znixian/OpenOVR) ·
  [DevOculus-Meta-Quest fork](https://github.com/DevOculus-Meta-Quest/OpenComposite/blob/openxr/README.md)
- OpenXR Toolkit — [mbucchia.github.io/OpenXR-Toolkit](https://mbucchia.github.io/OpenXR-Toolkit/)
- VRto3D — [github.com/oneup03/VRto3D](https://github.com/oneup03/VRto3D) ·
  [Helix Mod: VRto3D Driver](https://helixmod.blogspot.com/2024/07/vrto3d-driver.html)
- Depth3D — [github.com/BlueSkyDefender/Depth3D](https://github.com/BlueSkyDefender/Depth3D)
- ReShade (crosire) — [reshade.me](https://reshade.me/)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
