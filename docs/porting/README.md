# Porting a new engine: the adapter model + 10-milestone checklist

This is the public method for adding 6DoF VR to an engine that has **no** turnkey tool. It is
distilled from `elliotttate/vrframework` (derived from praydog/REFramework) and the two mutars
adapters (`starfield2vr`, `anvilengine2vr`). All credit in
[`../../ATTRIBUTION.md`](../../ATTRIBUTION.md); consult those repos as the ground truth before
writing code.

## The three-layer model

- **Layer 1 — Universal Core** (reused, never modified): D3D hooks, ImGui overlay,
  OpenVR/OpenXR runtime, stereo submission, config, pattern-scan/memory utilities. See
  [engine-agnostic core](../engine-agnostic-core/).
- **Layer 2 — Per-engine adapter** (what you write): one class implementing the engine interface.
- **Layer 3 — Per-game data**: offset manifests, reclass structs, per-title settings.

## The adapter interface (`IEngineAdapter`, from vrframework)

The whole per-engine contract is a handful of methods. Pure-virtual (must implement):

- `capabilities()` → declare the engine's feature set + **matrix basis / handedness** +
  submission mode (e.g. AFR).
- `install_hooks()` → resolve addresses (pattern scan / manifest) and install the engine hooks;
  return failure to abort init.
- `timeline()` → expose your `FrameTimeline` (frame counters + a skip-present flag) to the core.
- `apply_stereo(const StereoView&)` → the core hands you ready per-eye view+projection matrices
  (already basis-converted); you write them wherever the engine keeps its camera. Called per
  eye/frame.

Optional overrides: `get_world_camera()`, `reproject_hud(scale_x, scale_y)`,
`on_engine_input()`, `disable_incompatible_effects()`.

Supporting types: `EngineCaps` (flags + basis + a `has_taa` flag + `Submission` mode),
`StereoView` (per-eye matrices), `FrameTimeline` (events `ENGINE_FRAME_BEGIN`, `WAIT_RENDER`,
`RENDER_FRAME_BEGIN`, `PRESENT_BEGIN`, `PRESENT_END`; `is_left_eye_frame()`, `report(...)`,
`wants_skip_present()`; drives `on_wait_rendering` / `on_begin_rendering` / `on_update_hmd_state`).

## The 10 milestones (verify each before the next)

The method is **milestone-driven**: each step must produce a verifiable result before you build
on it. Skipping verification hides bugs downstream.

1. **Inject + draw an ImGui overlay.** Proxy `dxgi.dll` beside the exe; register your UI.
   *Verify:* the framework menu + your adapter section appear. *Fail:* wrong proxy name,
   32/64-bit mismatch, anti-cheat, no window-message hook.
2. **Hook Present; recover device + queue.** On D3D12 the command queue must be sniffed from the
   `ExecuteCommandLists`/present path (not reachable from the swapchain). *Verify:* stable
   non-null pointers each frame; cross-check in RenderDoc/PIX.
3. **Read engine view + projection.** Find the world→view and projection functions (usually two).
   Converge pattern-scan + graphics-debugger cbuffer inspection + known-value memory scan.
   *Verify:* matrices respond to movement; **document handedness/basis**. *Fail:* hooked a
   shadow/reflection camera; one-frame lag; row/column-major transpose.
4. **Get HMD poses from the runtime.** The core owns OpenVR/OpenXR init. *Verify:* pose tracks
   the head; per-eye projections are **asymmetric** (off-center frustums = correct).
5. **Inject one eye.** Compose
   `basis_change × rotation_offset × hmd_transform × eye_transform × inverse_basis_change`,
   replace projection with the runtime frustum, write via `apply_stereo()`. *Verify:* the mono
   image is head-tracked. *Fail (dominant):* world swims → basis/handedness wrong; black screen →
   gate on HMD-active and not-showing-UI; double-vision → you hooked before the matrix is
   recomputed.
6. **AFR both eyes.** Alternate the eye by frame parity; per-eye framerate = half engine
   framerate. **Temporal effects (TAA, motion vectors, history) break under AFR** — either
   double-buffer past-frame state per eye or disable the effect. See
   [techniques/taa-under-afr](../techniques/). *Signal you're here:* stereo but smeared.
7. **Fix frame timing (the hard one).** Map engine events to the `FrameTimeline` events and drive
   the three VR callbacks. Two proven signal sources: **two-hook style** (hook begin-engine-frame
   + begin-render-frame) and **Reflex-marker style** (decode existing Reflex markers). See
   [techniques/frame-timing](../techniques/). *Reality:* usually takes longer than all the others
   combined.
8. **HUD.** Expand scissor/viewport to the full eye RT, then rescale UI with user-tunable factors;
   suppress the stereo override during UI. *Verify:* centered, readable, unclipped, live sliders,
   no nausea.
9. **Input.** Overlay control from the controller + engine input remap. *Verify:* controller
   drives overlay and gameplay; aim follows controller if implemented.
10. **Tweaks.** Comfort/fidelity passes: decoupled pitch, world scale/IPD, disable incompatible
    effects (vignette, DOF, motion blur, letterbox), per-title config persistence.

## The irreducibly hard parts (called out by the public guides)

- **Finding stripped functions** across game updates — offset manifests need permanent
  maintenance.
- **Frame timing (M7)** — a subtle drift-recovery problem; each engine exposes different signals.
- **Temporal effects under AFR (M6)** — every temporal feature needs per-eye handling.
- **Closed engines render once per frame** — you cannot force native stereo; AFR's halved per-eye
  framerate is structural (the same wall [AER](../techniques/) addresses).

## Key principles

Never skip a milestone · four responsibilities = one interface · **read (log matrices, M3) before
write (M5)** · lean on public worked examples ([case studies](../case-studies/)) · defer comfort
tweaks to the end.
