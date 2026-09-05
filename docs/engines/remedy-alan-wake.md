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
- **⚠️ A plain `IDirect3D9::CreateDevice` vtable hook (slot 16) reliably breaks startup here** —
  `[verified-live 2026-08-25, n=1]`, and the **"cause unknown"** that stood beside it for ten days is
  now `[disproved 2026-09-04]`. The cause is mechanical and needed no experiment, only a careful read
  of a lifetime: the hook wrote an address **inside the proxy DLL** into slot 16 and **nothing ever
  restored the original**, while this game unloads the proxy about **6 ms** after the creation call
  (below). A D3D9 vtable is shared per interface class, so the patch outlived every object it was
  reached through, and the next `CreateDevice` jumped into unmapped memory — precisely the reported
  access violation, every time. **Nothing about this engine or title is special**, which is the part
  worth carrying: the same pattern works elsewhere in this account only because those proxies are not
  unloaded mid-run. An unhook path now exists and runs before the module is released; the hook stays
  **deliberately disabled** until a launch of its own tests it. The general form is on the techniques
  page as
  [a vtable patch is a lifetime commitment](../techniques/README.md#a-vtable-patch-is-a-lifetime-commitment--restore-it-before-anything-can-unload-you).
  The original detour is still instructive — the hook had been added *to investigate* an earlier crash
  and became the crash; see
  [the instrument can be the bug](../techniques/#the-instrument-can-be-the-bug). ⚠️ **A "confirmed
  broken, cause unknown" verdict on a standard technique should read as an open question**; labelling
  it a mystery is what kept anyone from looking for ten days.
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

### 2026-09-04: this game unloads and reloads D3D9 — so a proxy that never frees the real DLL is bypassed

`[measured 2026-09-04, n=3 launches]` The dynamic-resolution bullet above has a consequence nobody on
this family should have to rediscover. Alan Wake loads `d3d9.dll`, calls the creation export **once**,
unloads it about **6 ms** later, and then loads `"d3d9.dll"` a **second** time for the device it
actually renders with. Because Windows resolves a bare module name to any module already resident under that
base name, a proxy that loaded the system `d3d9.dll` by full path and never released it on
`DLL_PROCESS_DETACH` **leaves the system copy resident, and the game's second load binds to that** —
the game folder is never searched again and the proxy never runs a second time.

The tell is a proxy log whose entire content per launch is load, one export call, unload within about
100 ms, while the game reaches gameplay normally. Reads as a crash; is not one.

**This is old, known, and fixed the same way once before.** ReShade carried the identical defect
against this exact game until commit
[`74347b91d`](https://github.com/crosire/reshade/commit/74347b91d7729a6da93040298c6587bb3b786da4)
(2019-12-19, shipped in 4.5.2) — titled *"Fix hooking in Alan Wake"*, with the diff's own comment
saying that freeing the reference to the module loaded for export hooks *"is necessary for Alan Wake to
work"*. Seven years later, a from-scratch proxy on the same game met the same wall for the same reason.
**✅ FIXED AND CONFIRMED LIVE 2026-09-04** `[verified-live 2026-09-04, n=1 launch]`. Releasing the
system module on detach — guarded so it only happens on a real `FreeLibrary` and not during process
teardown — put the proxy back in the chain: one process id now shows the probe load, the creation call,
an explicit-`FreeLibrary` unload, **a second "proxy loaded" block with the same process id**, and then
no further unload for the session, with the title screen and menu both rendering through it. **For this
project that is the central unblock** — for weeks the proxy had only ever seen the throwaway probe
device, and it now owns the device the game actually renders with, so device-level interception is
finally reachable. The loader-lock caveat that comes with calling `FreeLibrary` from `DllMain` did not
bite on that launch, which is one data point rather than a clearance.

The engine-agnostic form, the loader quote and the estate audit are on the techniques page:
[a proxy must free the real DLL on detach](../techniques/README.md#and-it-must-free-the-real-dll-on-detach-or-a-reload-walks-straight-past-it).

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

### ⭐⭐ 2026-09-05: the handedness question is answered LIVE, and the constant traffic has a shape worth knowing

`[verified-live 2026-09-05, n=1 launch, 10 five-second windows per register]` The assumption the
entire static line above rested on — that `g_mViewToClip` is **left-handed with `clip.w = +view.z`**
— is now measured rather than assumed. Three true projections were seen uploaded during the
in-engine intro, all three with the w-from-`z` entry at **+1**, at the storage index implied by the
`MATRIX_ROWS` convention the shader metadata had already settled. The per-eye maths written against
that assumption stands unchanged, which is the good outcome and not the interesting one.

**Two things of engine-shaped value came out of the same launches:**

- **This engine uploads shader constants in whole 128-register blocks.** Roughly 6,000 uploads each
  of `c0+128` and `c128+128` per second in a 3D scene, plus tiny `c0+4` / `c0+5` uploads for the
  video quad and UI. **Any per-eye edit must locate its matrix by offset *inside* a block** — a hook
  that waits for an upload whose start register equals the register of interest will never fire, and
  a candidate loop that stops at the first spanned register will only ever see `c0`. Both registers
  this engine puts the projection in (`c0` or `c192`, per the skinning-palette displacement above)
  land mid-block. The three projections seen were the main camera at `c192` (16:9, near 0.2 / far
  1000, horizontal FOV 79.2°, vertical 50.0°), the same lens at `c7` on a far depth slice
  (1068 … 10000, distant scenery and sky), and a square 90° shadow or cubemap-face projection at
  `c0`.
- **Both hooks survive real frames**, through the intro and out to a clean menu quit, unhooked in
  order — but one launch in four **crashed on a layered-hook race** created by this game's
  probe-and-reload behaviour documented above. Detail and the rules that prevent it:
  [the slot you chain into may not be the engine's](../techniques/README.md#-and-the-reload-the-fix-enables-has-a-race-of-its-own-the-slot-you-chain-into-may-not-be-the-engines).
  The two findings are directly connected — freeing the real DLL is what lets the game load the proxy
  a second time, and the second load is where the race lives.

The instrument built the day before could not have seen any of this, for two reasons worth reading if
you are writing one:
[an instrument that tests one convention can only ever report "neither"](../techniques/README.md#an-instrument-that-tests-one-convention-can-only-ever-report-neither-convention).
Still open on this engine: which unit the eye separation should be expressed in, and whether a second
path rewrites those registers after a proxy does.

## See also

- [engines index](../engines-index.md) — the "Bespoke / older custom engines" row.
