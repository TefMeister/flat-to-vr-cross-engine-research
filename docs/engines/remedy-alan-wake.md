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

## See also

- [engines index](../engines-index.md) — the "Bespoke / older custom engines" row.
