# id Tech 6

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** id Software's id Tech 6 (DOOM 2016). Source not released.
- **Render API:** OpenGL **or** Vulkan — an exe-level fork: one shipped binary imports
  `OPENGL32`, the other `vulkan-1`.
- **Known public VR path:** none turnkey (Vk3DVision offers stereoscopic 3D only, not 6DoF).
  The engine ships a **dormant inherited stereo-3D path** (`stereoRenderMode_t`,
  `stereoRender_*`) and a named renderparm table — but retail production mode never registers
  the stereo cvars, so injection is the only route. Z-up basis; view angles as pitch/yaw degrees.

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| DOOM (2016) | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/doom-2016-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr) |

## Shared findings

*Seeded 2026-08-26; first populated 2026-09-01. Only one project sits on this family so far, so
everything here is `n=1` by construction — it is recorded as family truth because it is
engine-level rather than title-level, not because it has been seen twice.*

### The camera

- **The binary names its own camera.** id Tech 6 exposes shader constants as named **renderparms**
  and ships the whole name table as plain strings: `viewMatrixX/Y/Z/W`, `projectionMatrix*`,
  `mvpMatrix*`, and the quartet **`globalViewOrigin` / `globalViewFwd` / `globalViewLeft` /
  `globalViewUp`**. Matrices arrive as four separate vec4 renderparms, so override granularity is
  per-row. `[inferred-static 2026-08-26]`
- **Basis and convention:** **Z-up**, classic id/Quake lineage; view angles in degrees 0–360; roll
  not exposed for the player view. The console prints the live camera, which makes it a permanent
  ground-truth instrument. `[verified-live 2026-08-26]` **Careful with the print order** — on DOOM
  2016 it is **yaw then pitch**, and a swapped pair produces a plausible-looking but wrong camera.
  `[verified-live 2026-08-31, derived from the matrix]`
- **The authoritative view lives in a single static global, not on the heap.** On DOOM 2016 that is
  twelve contiguous floats — origin plus a full orthonormal basis, the renderparm quartet above — in
  the executable's **image** region. The per-draw copies visible in GPU-mapped memory are downstream
  replicas, replicated dozens of times per frame. `[verified-live 2026-09-01, n=1 process instance]`
- **Culling follows the camera, for free.** Displacing the view origin renders the world correctly
  from a position the player is not at — no culling collapse, no black void. That is not luck: the
  engine ships a **detached photo-mode camera**, so the path was designed for it. This is precisely
  what other engine families in this library have spent weeks failing to get.
- **Uniform delivery is CPU-written host-visible memory.** DOOM 2016 imports neither
  `vkCmdPushConstants` nor `vkCmdUpdateBuffer`, so every uniform arrives through mapped memory the
  CPU writes directly. **But the camera buffer is `HOST_COHERENT`**, so `vkFlushMappedMemoryRanges`
  never carries it and is the wrong interception point despite being the obvious one — the queue
  submit is the real gate. `[measured 2026-08-31]`

### Stereo

- The family carries a **dormant inherited stereo-3D path** (`stereoRenderMode_t`, `stereoRender_*`),
  traceable to the Doom 3 BFG generation and to Carmack's own 2012 Rift work. Retail production mode
  never registers those cvars.
- **⭐ There is no stereo *mode* cvar to find — the switch is a call argument.** Two independent
  negative reads settle it: the **published 6,572-cvar dump** contains all four `stereoRender_*`
  parameters, `multiView_60Hz` and `com_production`, but **nothing selecting `stereoRenderMode_t`**
  `[reported 2026-09-01]`; and retail registers only 171 cvars at runtime, with `listCvars stereo`
  returning nothing `[verified-live 2026-08-26]`. id's published GPL source for the previous
  generation threads the eye through the render call instead — `RB_DrawView(data, stereoEye)` with
  `0` mono and `±1` per eye, plus a first-class `viewEyeBuffer` field on the view object.
  `[reported 2026-09-01]` for id Tech 4/5, **`[hypothesis]` for id Tech 6** — but it *explains the
  measured cvar inventory*, which is what lifts it above a guess: every stereo **parameter** exposed
  while no **mode** is, is exactly what a call-site argument looks like from outside.
  `stereoRender_swapEyes` is only a late cosmetic flip, not the gate.
  - **Consequence, and it removes work:** winning the console/production-mode gate would yield the
    stereo *parameters*, **not the on-switch** — so the gate is off the critical path for stereo.
    Hunt instead for a function taking a small signed eye argument called twice per frame, and for
    the matching field on the view object; the reflection database below makes that a **static**
    search needing no launch. Generalised on the
    [techniques page](../techniques/#the-switch-you-cannot-find-may-be-an-argument-not-a-global).
- **Separation is applied at the view stage as well as the projection stage.** id's own GPL source
  for the previous generation declares the view origin as *already adjusted for stereo world
  separation*, with a separate *screen* separation term shifting the projection horizontally — which
  is why two separation cvars exist. See the case study for the full correction and the tension it
  leaves open. **Practical consequence for this family: try the view stage first.**
- **Public prior art is stereo-only on id Tech 6.** The one Vulkan stereo driver with a DOOM 2016 fix
  provides no head tracking, and its repository was archived 2026-03-05. The 6DoF VR package from the
  same author exists only for **id Tech 7 / DOOM Eternal**, built on single-pass stereo instancing.
- **TAA with jittered projection and motion-vector reprojection is present** (`mvpMatrixNoJitter*`
  alongside `mvpMatrixLast*`) — a known VR hazard, and spotted statically before ever launching.

### Getting in, and getting the console

- **Injection is over-supplied and unprotected.** No packing; ASLR and Control Flow Guard on. The
  renderer choice is an **executable-level fork** — one binary imports `OPENGL32`, the other
  `vulkan-1` — and the Vulkan build imports ~96 entry points **statically and directly**, with no
  `vkGetInstanceProcAddr` / `vkGetDeviceProcAddr`, so a plain `vulkan-1` proxy sees all Vulkan
  traffic with no dispatch-table funnel. `dinput8` and `winmm` are imported in both.
- **Retail boots into production mode**, registering a small fraction of the console vocabulary, with
  the master switch itself unregistered. **But a public community unlocker exists** and re-adds the
  hidden interface without developer mode — including a renderparm read/write command and a
  set-view-position command. See
  [Before you build it, check whether the game shipped it](../techniques/#before-you-build-it-check-whether-the-game-shipped-it).
- **A reflection database with the developers' own doc-comments ships in the binary** — fully
  qualified C++ class, enum and field names next to human-written descriptions. Functionally a
  built-in symbol source, the way reflection serves UEVR on Unreal or REFramework on RE Engine,
  except native to the file. Mining it properly is its own worthwhile task.
- **Named override-shaped fields already exist — but they are not cvars.** `explicitProjectionMatrix`,
  `explicitFov_x` / `explicitFov_y`, `forceIdentityViewMatrix`. If honoured on the main world view, a
  per-eye projection override could be a *supported engine input* rather than a patch — still
  unverified. **Refined 2026-09-01:** the full cvar-dump read found **none of them present as cvars**
  (every `explicit*` hit belongs to `ai_`, `pm_`, `fs_` or `prowler_`) `[reported 2026-09-01]`, so
  they are **renderparms or code-level fields**, and the route to them is the renderparm command
  (`rp`) or a patch — not the console variable system. Worth knowing before a session is spent
  typing them at a prompt.

### Input

**Not raw input.** DirectInput 8 in **non-exclusive** mode for gameplay keyboard and mouse, XInput
1.4 linked directly, Win32 key state for menus and text entry, and centre-the-cursor mouse-look via
`Get`/`SetCursorPos`. `SendInput` therefore drives the game completely, with the game foregrounded;
in-process key-state hooks install perfectly and do nothing. `[measured + verified-live 2026-08-31]`
Full detail and the general rule:
[injected input](../techniques/#injected-input-measure-it-against-a-control-never-against-zero).


**Eye-field hunt, 2026-09-03.** The BFG-source prediction that `viewEyeBuffer` and
`stereoScreenSeparation` would be adjacent reflection fields was tested and **disproved for id Tech
6** — zero hits in either exe, exhaustive substring search `[disproved 2026-09-03, n=2 exes]`. The
look-alike pair `leftFrameOffset`/`rightFrameOffset` is adjacent in the reflection table but types as
two 256-byte buffers, not floats. Names transfer across id generations; layouts do not. No positive
eye-field candidate exists; walking the 72-byte reflection records outward from a known anchor is the
untried static angle. The `ringcam` write path remains compile-verified and never run.

### 2026-09-04: a virtual XInput pad drives this engine, and a value-scan taught a lesson about its base

- **⭐ A ViGEmBus virtual Xbox 360 pad is bound as a real controller and drives both movement and
  look** `[verified-live 2026-09-04, n=2 per axis with reversal]`. Left stick: pure translation, camera
  basis unchanged. Right stick: pure yaw, position unchanged. Both reversed cleanly, and destroying the
  pad raised the game's own *"Controller Disconnected"* toast — confirmation the game had bound the
  device rather than merely that Windows enumerated it. This engine links XInput 1.4 directly, which is
  the precondition. **It is focus-independent**, unlike the `SendInput` look route recorded above, so it
  is the better instrument for camera work on this family. Cross-engine write-up:
  [known input routes](../techniques/README.md#known-input-routes-by-engine-family).
- **A camera-copy scan found zero, and the cap was a red herring** `[verified-live 2026-09-04]`. The
  scanner's window was genuinely 6.6× too small and that was genuinely fixed — and the widened scan
  still found nothing, because it was scanning the process's *largest* mapping while a by-value search
  located the sixty-four camera copies clustered near the start of a **different** region. Base, not
  range. The general form is on the techniques page:
  [check the base before you widen the range](../techniques/README.md#when-a-scan-finds-nothing-check-its-base-before-you-widen-its-range).
  One loose end for this family: the region actually holding the copies reports tens of thousands of
  flushes, which sits oddly beside this page's reading that the camera buffer is host-coherent and off
  the flush path — worth resolving before either statement is relied on.

## See also

- [engines index](../engines-index.md) — the "id Tech 6" row.
- [Case study: id Tech 6's dormant stereo path](../case-studies/id-tech-6-dormant-stereo.md) —
  including the `strings -n` method trap that produced (and then corrected) a wrong conclusion.
- [id Tech 5](./id-tech-5.md) — the previous generation.
