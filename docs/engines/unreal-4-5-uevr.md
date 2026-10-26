# Unreal Engine 4–5 (through UEVR)

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep. Page started 2026-10-26.*

## Identity

- **Engine:** Epic's Unreal Engine 4.8 to 5.x. The older generations (UE1–3) have their own page,
  [unreal-1-3.md](./unreal-1-3.md); UEVR does not reach them.
- **Render API:** Direct3D 11 or 12.
- **Known public VR path:** **praydog's UEVR**, a turnkey injector that drives the engine's own stereo
  through its reflection system. Most UE4/5 VR work, ours included, is a layer on top of UEVR: community
  Lua profiles, C++ plugins, or both. PureDark's fork adds frame warping
  (see [runtime layers](../runtime-layers/README.md#frame-warping-as-a-plug-in-puredarks-afw)).

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| Silent Hill 2 (2024 remake), UE 5.1 — a native C++ UEVR plugin beside praydog's own plugin and a community Lua pack | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/silent-hill-2-remake-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`silent-hill-2-remake-vr`](https://github.com/TefMeister/silent-hill-2-remake-vr) |

## Shared findings

*Everything below is from one project so far (`n=1` by construction), but each finding is about UEVR or
the engine, not about the game.*

### C++ plugins edit the pad before Lua, and Lua always edits last

Read in UEVR's public source by our `/gr` lane (praydog's master and PureDark's AFW fork carry the same
lines) `[inferred-static 2026-10-26]`:

- Every pad callback, from a C++ plugin or from Lua, sits in **one list** that each pad poll walks front to
  back, each callback editing the same controller state. Whoever registered last has the final word.
- **C++ plugins register at start-up**, in **file-name order** (the loader keeps plugins in a sorted map).
- **Lua registers on UEVR's first frame, and is removed and re-added at the end on every "Reset
  scripts".** So a C++ plugin cannot stay after Lua by load order alone.

So on any UEVR game: **C++ sees the real controller first, Lua has the last word.** A project that ports a
community Lua profile to C++ piece by piece will meet "one press does two things" while both still write
the pad; the clean fix is to switch the Lua profile's pad writes off, not to try to undo them from C++.
Corroborated live: the UEVR MCP plugin's simulated pad (file name sorting after ours) was applied after our
plugin's callback, so our plugin never saw it `[verified-live 2026-10-24]`.
Write-up: `silent-hill-2-remake-vr/external-research/topics/2026-10-26-uevr-runs-cpp-plugins-before-lua-on-the-pad.md`.

### Running it with no headset

UEVR attached with the headset runtime idle failed to start its D3D12 framework every frame, and the game's
first screen took no input. Starting the game with `XR_RUNTIME_JSON` pointing at the OpenXR Simulator's
manifest ran it in VR on the simulator `[verified-live 2026-10-24, n=3]`. Starting a Steam game from its
exe needs the Steam app id in the environment, or the save profile is missing; see
[techniques](../techniques/README.md#launching-a-steamworks-game-directly).

### Reading a shipping exe with no RTTI

Two static methods that worked on a 98 MB UE5 shipping build `[inferred-static 2026-10-24]`:

- **UFunction name → native wrapper:** the engine keeps `{name pointer, exec function}` pairs in read-only
  data. Find the pointer to the UTF-8 name; the next pointer is the wrapper. Two tables point at each name;
  the pair table is the one whose neighbours are code addresses near each other.
- **Class → vtable:** the class name (UTF-16) is referenced in its `StaticClass()`; the registration call
  there passes the class's internal constructor, which jumps to the real constructor, whose first store of
  an address into the object is the vtable.

Reflected property **names** by offset were not recovered this way; naming them live through the UEVR
MCP's object inspection was quicker.

### Two traps when steering gameplay from a plugin

- **Forcing a UFunction's answer does not reach native callers.** A `IsFireReady` UFunction is only a
  wrapper round a virtual; the game's own fire path calls the virtual directly, so forcing the UFunction to
  true changed nothing `[verified-live 2026-10-24, n=1]`. Find the virtual before hooking the wrapper.
- **Event hooks go on the object's own Blueprint class.** A hook on the base class's UFunction never fired
  for `OnFirePerformed`, because the weapon's Blueprint class overrides it; hooking the Blueprint class's
  function did fire `[verified-live 2026-10-24, n=1]`.

### Ask the running engine

UEVR-MCP (elliotttate) reads the engine's reflection live: objects, fields, calls, hooks and a simulated
pad. See [techniques](../techniques/README.md) → "Ask the running engine itself".

## See also

- [engines index](../engines-index.md) — the "Unreal Engine 4.8 – 5.x" row.
- [Unreal Engine 1–3](./unreal-1-3.md) — the older generations, where UEVR does not reach.
- [runtime layers](../runtime-layers/README.md) — PureDark's frame warping and the OpenXR Simulator.
