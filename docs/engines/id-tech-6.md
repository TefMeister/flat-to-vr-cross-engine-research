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
- **⭐⭐ 2026-09-05 — the eye is an INDEX ON THE VIEW OBJECT, not a field on the camera, and the
  reflection database says so in the developers' own words.** The whole field table was walked:
  **4,774 classes**, **57,214 field records** (99.98% of the population; a deliberately looser
  validator returned **+0** more), with the class layer self-checking against three independently
  known `sizeof` values. `[verified-numerically 2026-09-05]`
  - **The negative, and it is near-exhaustive.** **Zero** field names contain `stereo`; all 59
    occurrences of `eye` are gameplay, AI or animation. Every view-shaped class was enumerated in
    full — the render view struct (61 fields), the view object (110), the render view (35), the
    screen view (8), the view bypass (6) and the frame info (4) — and **none carries a per-eye
    field.** Positive control: the same scan re-found the per-frame offset pair, the
    explicit-projection-matrix override and its boolean, the FOV pair, the near-clip clamp, and the
    view origin and axis, unprompted.
  - **The positive is in the doc comments, not the names.** The database stores id's own comments,
    and searching *those* returns six hits, five renderer: a **`viewIndex`** on the screen view
    *("determines which viewColor image will be rendered to, and which view from world will be
    used")*; a **fixed-capacity list of two** world views on the frame info *("two identical ones in
    stereo-3D (both centered between the eyes)")*, with **capacity 2 compiled into retail**, confirmed
    from the container's own byte size; a screen-view list whose comment says *"stereo-3D will define
    two views"*; and — the structural finding — a **GUI** origin offset *("for stereo 3D, the guis can
    be offset differently in each screenView")* that is **the only surviving per-view stereo scalar,
    with no world-camera counterpart.** That is why the previous generation's per-eye view field was
    never going to be found here: it was replaced by *which view you are rendering*, not moved.
  - **So the route to test is populating the second world-view entry**, which is
    `[inferred-static 2026-09-05]` — no code path has been traced to it.
  - **A latch to keep on file:** the render view holds a game-set copy and a *renderer* copy
    described as *"latched at EndFrame time"*, so a write landing after that point is discarded for
    the frame. `[hypothesis]` as an explanation of anything observed so far; do not use it as a
    diagnosis until something is measured against it.
  - **Two limits, stated because they bound the claim.** The tables list **reflected** members only,
    so the negative is exact for *"is there a reflected eye field"* and only strongly suggestive for
    *"is there an eye field"*. And the tables have **no code references at all** — neither absolute
    nor RIP-relative, checked at five addresses — so they map the data completely and give **no
    static anchor into renderer code**.
  - The method is engine-agnostic and is written up as
    [search the doc comments, not the names](../techniques/README.md#-a-reflection-table-often-carries-the-developers-own-doc-comments--search-those-not-the-names);
    the cross-reference caveat is
    [the ModRM hole](../techniques/README.md#-a-cross-reference-scanner-that-does-not-decode-modrm-is-blind-on-x64--and-every-no-xrefs-result-it-produced-is-suspect).

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
  **✅ The loose end that entry raised is closed the same day, and the answer is a reasoning error
  rather than a fact about the engine.** The region holding the copies reported tens of thousands of
  `vkFlushMappedMemoryRanges` calls, which looked like it contradicted this page's reading that the
  camera buffer is host-coherent and therefore not updated through the flush path. It does not. The
  Vulkan specification says those cache-management commands **are not needed** on coherent memory, and
  *not needed* is not *not allowed* — an engine that flushes unconditionally, without branching on
  memory type, produces exactly that count while the flush does no work `[reported 2026-09-04]`. **The
  count was compatible with both readings, so it was never evidence for either.** The discriminator is
  one field, the memory type's property flags. General form:
  [a legal-but-unnecessary call is not evidence of a mechanism](../techniques/README.md#-the-inverse-a-legal-but-unnecessary-call-is-not-evidence-of-a-mechanism).

## See also

- [engines index](../engines-index.md) — the "id Tech 6" row.
- [Case study: id Tech 6's dormant stereo path](../case-studies/id-tech-6-dormant-stereo.md) —
  including the `strings -n` method trap that produced (and then corrected) a wrong conclusion.
- [id Tech 5](./id-tech-5.md) — the previous generation.
