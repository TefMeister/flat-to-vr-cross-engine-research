# RenderWare

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** Criterion Software's RenderWare — the widely licensed 2000s middleware engine
  (used across many publishers' titles of the era).
- **Render API:** Direct3D 8/9 era, **fixed-function** — the PC builds of this generation predate
  shader-based pipelines, so there is no constant buffer to hunt: the view and projection matrices
  reach the GPU through `IDirect3DDevice8::SetTransform`.
- **Known public VR path:** none turnkey, and none by injection. There *is* real VR prior art on this
  family, but it arrives by a route most projects here cannot take — see "the prior art, and what it
  does not transfer" below. For anything Direct3D 8 or older, a D3D8-to-modern wrapper comes first
  (see the engines index's last row).

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| Manhunt (2003) | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/manhunt-2003-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`manhunt-2003-vr`](https://github.com/TefMeister/manhunt-2003-vr) |

## Shared findings

*Seeded 2026-08-26; first populated 2026-09-01 from Manhunt's dossier plus one public source, so
everything below is `n=1` by construction.*

### The prior art, and what it does not transfer

`[reported 2026-09-01]` The family's only substantial VR conversion is
[**Vice City VR**](https://github.com/dubrovskiy-yevhen-stakelogic/vice-city-vr) — an unofficial
6DoF OpenXR conversion of the 2003 PC release of GTA: Vice City, publicly active as of 2026-08-31,
with a [native Quest sibling](https://github.com/Blackbird88/vice-city-vr-quest). Its own description
is the important part: it is built on a **reverse-engineered source reimplementation** of the game
plus [**librw**](https://github.com/aap/librw) (aap, MIT — an open-source reimplementation of the
RenderWare graphics engine), and it **replaces the graphics pipeline outright** with Direct3D 12,
single-pass stereo, variable-rate-shading foveation and modern upscaling. Only the release is public;
the runtime source is private during development.

**So the existence proof is real, and its method is not available to a typical project on this
family.** It is a *source port*, not an injection: it needs a full decompilation of that specific
game, and it ships a different renderer rather than hooking the shipped one. Two consequences:

- For a RenderWare title with no decompilation, this says nothing encouraging about difficulty and
  nothing at all about method. The injection route stands on its own.
- The underlying reimplementation projects for the GTA III/Vice City pair are the subject of a
  publisher takedown and the original repository **returns HTTP 451 (access blocked)** on GitHub as
  of 2026-09-01. `librw` itself is a separate, unaffected MIT project. This library does not clone or
  study anyone's code regardless (see [`CONTRIBUTING.md`](../../CONTRIBUTING.md)), but it is worth
  recording so that nobody plans work around a dependency that may not be there.

### The injection route on this family

- **The camera lever is `SetTransform`, not a constant buffer.** `[inferred-static]` A fixed-function
  D3D8 engine hands `D3DTS_WORLD` / `D3DTS_VIEW` / `D3DTS_PROJECTION` to the device directly, so both
  halves of stereo — a per-eye view translation and an asymmetric per-eye frustum — sit at one call.
  The same shape as the UE2 finding on the [Unreal 1–3 page](./unreal-1-3.md), and for the same
  reason: both are D3D8 fixed-function engines of the same years. **Check for a cached-matrix
  early-out before writing the hook**; see
  [the stereo hazard](../techniques/#stereo-hazard-a-setter-that-early-outs-on-an-unchanged-matrix).
- **Expect a live protector stub, and expect it to invalidate static analysis.** Manhunt's retail
  build has its entry point inside an oversized `.bind` section and blocks external debugger attach
  entirely, while a same-name DLL proxy loads and works normally. Full treatment:
  [packed/self-protecting binaries](../techniques/#packedself-protecting-binaries) and the
  [case study](../case-studies/packed-binary-live-memory-scan.md).
- **A stripped licensing layer can leave sabotage behind that reads as engine bugs.** Sixteen call
  sites in that build each expected a specific faked value from a protection stub that no longer
  exists, and the game damages itself on each failure path. Symptoms — stuck doors, broken saves,
  erratic AI, crashes on item swap — will be blamed on your mod unless you prove otherwise; see
  [remove your own code before accepting the blame](../techniques/#remove-your-own-code-before-accepting-the-blame--then-fix-the-producer).
- **Video mode may be a registry value rather than a code path**, which makes windowed-mode work for
  a fullscreen-only title a one-DWORD change rather than a device-creation fight. See
  [the setting you want to change may be data, not code](../techniques/#the-setting-you-want-to-change-may-be-data-not-code).
- **The engine validates its camera raster against the window's client area.** In Manhunt's
  RenderWare build the sized-raster path calls `GetClientRect` and refuses anything larger, with its
  own error string; the refused allocation returns NULL and that single NULL cascades into
  null-pointer crashes deep in unrelated engine code. Any retrofit that forces windowed mode on this
  family must resize the window before the engine builds its camera.

## See also

- [engines index](../engines-index.md) — the "Bespoke / older custom engines" and "Anything
  Direct3D 8 or older" rows.
- [Generic drivers for older D3D9 games](../generic-drivers/) — the vorpX/geo-11/dgVoodoo2
  routes relevant to this era.
