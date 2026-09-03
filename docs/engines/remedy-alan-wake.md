# Remedy's Alan Wake engine

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** Remedy Entertainment's proprietary in-house engine as shipped in Alan Wake — the
  studio lineage that later produced Northlight (Quantum Break onwards).
- **Render API:** **Direct3D 9**, 32-bit — but resolved dynamically rather than statically imported;
  see the shared findings below.
- **Known public VR path:** none turnkey. Manual build.

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| Alan Wake (PC) | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/alan-wake-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr) |

## Shared findings

*Seeded 2026-08-26; **first populated 2026-09-01** from the Alan Wake dossier, so `n=1` by
construction — this family has one project on it.*

- **The executable is a thin loader; the engine lives in per-subsystem DLLs.** `[inferred-static
  2026-08-25]` `AlanWake.exe` is a small PE32 with only four sections, and the real engine arrives as
  separately-named modules — `app_sf_Win32.dll`, `grph_sf_Win32.dll`, `d3d_sf_Win32.dll`,
  `renderer_sf_Win32.dll`, `physics_sf_Win32.dll`, `ai_sf_Win32.dll` and others, one per subsystem.
  **This is unusual for the era and it changes recon:** a string or import sweep of the executable
  alone finds almost nothing, and every question about the renderer has to be asked of
  `d3d_sf_Win32.dll` instead. Budget for sweeping *all* modules rather than the exe.
- **D3D9 is resolved dynamically, not statically imported.** `[inferred-static 2026-08-25]` No module
  imports `d3d9.dll` in its PE import table. `d3d_sf_Win32.dll` instead carries the strings
  `Direct3DCreate9` and `Direct3DCreate failed` side by side — the signature of
  `LoadLibrary` + `GetProcAddress` with a handled failure path. **A same-named proxy still works**
  (`LoadLibrary` uses the same application-directory-first search order), but the failure mode of an
  incomplete proxy is a graceful logged error rather than a silent pre-`main` exit. See
  [a proxy DLL must export everything the target actually imports](../techniques/#a-proxy-dll-must-export-everything-the-target-actually-imports).
- **A from-scratch `d3d9.dll` proxy is live-verified on this engine.** `[verified-live 2026-08-25,
  n=1]` The game runs cleanly with it and needs **no compatibility flag** of any kind.
- **⚠️ A plain `IDirect3D9::CreateDevice` vtable hook (slot 16) reliably breaks startup here, cause
  unknown.** `[verified-live 2026-08-25, n=1]` The same hook pattern works on sibling D3D9 projects in
  this account, so this is specific to this engine or title. It cost a full diagnostic detour, since
  the hook had been added *to investigate* an earlier crash and became the crash. Do not re-enable
  such a hook on this family without understanding it first — and see
  [the instrument can be the bug](../techniques/#the-instrument-can-be-the-bug).
- **Remedy shipped real developer tools in the retail build.** `[reported 2026-08-25]` `-freecamera`
  (a genuine free camera, toggled with the right thumbstick — controller only, no confirmed
  keyboard equivalent) and `-developermenu` (episode/difficulty/ammo; **not** confirmed to include
  camera or rendering tools). A zero-injection way to observe camera behaviour before hooking
  anything — the same category of asset as a dormant debug menu elsewhere in this account.
- **⚠️ The "native 3D Vision support" signal is weaker than it first reads.** `[reported 2026-08-25]`
  Later builds are described as close to "3D Vision ready out of the box", with separation adjustable
  in-game on `Ctrl+F3` / `Ctrl+F4`. **Corrected 2026-09-01:** those are the **driver's** hotkeys in
  3D Vision *Automatic* mode, where the driver appends a clip-space footer to each vertex shader and
  splits every draw itself. So this is evidence that 3D Vision worked *on* Alan Wake — **not** that
  the engine contains a native per-eye path, which is what an earlier reading of this bullet claimed.
  The static check that separates the two is which driver mode the binary requests
  (`NvAPI_Stereo_SetDriverMode`); see
  [the clip-space stereo footer](../techniques/#the-clip-space-stereo-footer-geometry-stereo-without-ever-finding-the-camera).
  Still useful, and arguably more so: the footer technique is implementable by our own proxy without
  NVIDIA anything.
- **No DRM, and the GFWL history is closed.** `[inferred-static 2026-08-25]` The original release
  shipped on Games for Windows Live; the installed Steam build has **zero** `xlive`/GFWL files or
  strings across the exe and all nine module DLLs. Checked specifically rather than assumed.

### §6 answered statically, 2026-09-03: projection arrives separately from view, and the camera is one global on a no-ASLR image

`[inferred-static 2026-09-03]` The shipped shader bank (`shaders/build/pc/`, 62 `RFX ` containers)
is **pre-compiled D3D9 bytecode with `CTAB` intact** — 9,971 constant tables, every constant named
with its register — so the off-disk reflection method transfers here after all, and the earlier
"the game compiles at runtime, proxy the compiler" reading is `[disproved 2026-09-03]`. What the
tables show is **the best matrix shape in the estate for stereo**: `g_mViewToClip` (projection, 4×4)
and `g_mLocalToView` (object-to-view, 4×3) are delivered **separately**, so per-eye rendering is two
independent single-float edits — eye translation into the view matrix, and a convergence shear
expressed through the projection's *own* `row0.x`, which keeps it correct when the game changes FOV.
Storage class is `MATRIX_ROWS`, settled two ways (metadata and `dp4` bytecode); the projection
register is **`c0` or `c192`** depending on a 192-register skinning palette, so a proxy must resolve
it per shader. 515 particle/foliage/terrain shaders bypass the split with a fused matrix and need the
clip-space form with `p00` supplied from a cached projection. All of it is verified numerically
(1,080 + 72 configurations, mutation-tested) and none of it has run in the game.

On the game side, a public cheat table's byte pattern (Jim2point0's, read online) ported exactly and
led to the camera object behind **one static global** with FOV at `+0x214`; **`AlanWake.exe` has no
ASLR** (`DYNAMIC_BASE` unset, no `.reloc`), so every static address on this project is permanent. The
critical path is now injection depth — device-level hooking of `SetVertexShaderConstantF` — not
knowledge. Engine-agnostic forms:
[the storage-class check](../techniques/README.md#determine-the-matrix-storage-class-two-ways-before-writing-any-per-eye-edit),
[the register-displacement trap](../techniques/README.md#the-register-is-not-fixed-a-skinning-palette-displaces-the-camera-constants),
and [the shipped-compiler correction](../techniques/README.md#when-a-game-compiles-its-shaders-decides-how-you-read-its-constant-map).
Also settled: the game is a 3D Vision **Automatic** consumer, and
[3D Vision itself is discontinued](../generic-drivers/README.md#-3d-vision-itself-is-discontinued--what-that-means-for-a-games-native-stereo-toggle).

## See also

- [engines index](../engines-index.md) — the "Bespoke / older custom engines" row.
