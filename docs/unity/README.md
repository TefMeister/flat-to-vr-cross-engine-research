# Unity games: the managed-runtime shortcut

**Unity is the one mainstream engine where flat→VR modding mostly skips the native
reverse-engineering layer entirely.** Unity games ship their game logic as .NET (Mono) or
IL2CPP-compiled C#, and the engine itself has first-class VR support built in (the XR plugin
system). A Unity VR mod therefore doesn't hook D3D or hunt for view matrices — it loads managed
code *inside* the game, flips the engine's own VR support on, and then works with real, named
`Camera`/`Transform` objects through the engine API.

That makes Unity the friendliest large target class in all of flat→VR modding — and the reason
it gets its own page instead of a row in the [engines index](../engines-index.md). All projects
credited in [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).

---

## Why Unity is different

- **Managed code = free reflection.** Mono-built games are inspectable and patchable at the C#
  level (dnSpy-class tooling, Harmony runtime patching) — no disassembly required.
- **The VR plumbing already exists.** Unity ships stereo rendering, head tracking, and input
  through its XR subsystem. A mod's job is to *activate and configure* it (or load the right XR
  plugin for the game's Unity version), not to build stereo from scratch.
- **A standard injection substrate exists.** **BepInEx** (and MelonLoader) are universal Unity
  plugin loaders — the mod is just a plugin DLL; the loader handles getting it into the process.
- **The catch: IL2CPP.** Games built with IL2CPP compile the C# away to native code. Modding is
  still possible (BepInEx/MelonLoader generate interop assemblies from the game's metadata), but
  everything is harder, and per-Unity-version XR-plugin matching gets fiddlier.

## The public tools

### UUVR — the universal Unity VR mod
**UUVR** (Raicuparta; GPL-3.0) is the generic "make this Unity game VR" mod — the Unity
counterpart to UEVR's role for Unreal. It targets a wide range of Unity versions across both
Mono and IL2CPP backends, and is installable via **Rai Pal** (same author; GPL-3.0), a mod
manager purpose-built for universal mods (it detects installed games, their engine, Unity
version and backend, and installs the right build). Expect headset rendering and head tracking
out of the box; deep motion-control gameplay still needs per-game work.
- [github.com/Raicuparta/uuvr](https://github.com/Raicuparta/uuvr) ·
  [Rai Pal](https://github.com/Raicuparta/rai-pal) · [raicuparta.com](https://raicuparta.com/)

### VRGIN — the classic injection framework
**VRGIN** (Eusth; MIT) is the older, influential VR injection framework for Unity: it manages
the cameras and GUI re-projection and provides a structure (controllers, standing/seated modes,
tools) for building a per-game VR mod on top. Injection itself is delegated to a plugin loader.
Its wiki page *"Hacking VR into a Unity game"* remains a good conceptual read for what a Unity
VR mod actually has to do.
- [github.com/Eusth/VRGIN](https://github.com/Eusth/VRGIN) ·
  [wiki: Hacking VR into a Unity game](https://github.com/Eusth/VRGIN/wiki/Hacking-VR-into-a-Unity-game)

### The per-game gold standard: Raicuparta's mods
Where UUVR is generic, these show what full per-game investment yields — complete VR games with
motion controls, made by editing game behavior at the C# level:

- **NomaiVR** — Outer Wilds VR, 6DoF + full motion controls (Raicuparta & artumino; MIT).
  [github.com/Raicuparta/nomai-vr](https://github.com/Raicuparta/nomai-vr)
- **TwoForksVR** — Firewatch VR: roomscale, motion controls, comfort options (Raicuparta; MIT).
  [github.com/Raicuparta/two-forks-vr](https://github.com/Raicuparta/two-forks-vr)

The scene has many more per-game Unity VR mods by many authors (Rai Pal's ecosystem and the
Flat2VR community index them); these two are called out for being mature, open (MIT), and
well-structured examples to *study* — as always, we learn the technique, not copy the code.

## Decision shortcut for a Unity target

1. Confirm it's Unity (folder layout: `<Game>_Data/`, `UnityPlayer.dll`; `globalgamemanagers`).
2. Identify **Unity version** and **Mono vs IL2CPP** (Rai Pal does this for you).
3. Try **UUVR** first — for many games it's minutes to a head-tracked stereo view.
4. For a real gameplay conversion, build a per-game BepInEx plugin; study NomaiVR/TwoForksVR
   for structure and VRGIN for the camera/GUI model.
5. Only if the game resists managed-level work (heavy IL2CPP obfuscation, ancient Unity) does it
   fall back to this library's native techniques ([porting](../porting/),
   [generic drivers](../generic-drivers/)).

## Sources

- UUVR — [github.com/Raicuparta/uuvr](https://github.com/Raicuparta/uuvr)
- Rai Pal — [github.com/Raicuparta/rai-pal](https://github.com/Raicuparta/rai-pal)
- Raicuparta's mods overview — [raicuparta.com](https://raicuparta.com/)
- VRGIN — [github.com/Eusth/VRGIN](https://github.com/Eusth/VRGIN)
- NomaiVR — [github.com/Raicuparta/nomai-vr](https://github.com/Raicuparta/nomai-vr)
- TwoForksVR — [github.com/Raicuparta/two-forks-vr](https://github.com/Raicuparta/two-forks-vr)
- BepInEx — [github.com/BepInEx/BepInEx](https://github.com/BepInEx/BepInEx) ·
  MelonLoader — [github.com/LavaGang/MelonLoader](https://github.com/LavaGang/MelonLoader)
- Harmony — [github.com/pardeike/Harmony](https://github.com/pardeike/Harmony)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
