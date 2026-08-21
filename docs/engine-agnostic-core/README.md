# The engine-agnostic core (the reusable half)

Every flat→VR mod splits cleanly into two halves:

- an **engine-agnostic half** that is essentially the same for every game, and
- an **engine-specific half** — finding and owning *this* engine's camera — that must be built
  from scratch each time.

This page is about the first half. The public injectors and frameworks (UEVR, REFramework,
vrframework and its derivatives) have already solved it, and their solutions are the reference.
Credit for all of it is in [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).

## What lives in the reusable core

Named after the way `elliotttate/vrframework` (derived from praydog/REFramework) organizes it —
its "Layer 1 / Universal Core":

- **Injection + framework lifecycle.** Get your code running in the target every frame, safely.
  The common vector is a **proxy DLL** (e.g. a `dxgi.dll` beside the game) that loads after any
  DRM has unwrapped in memory. Forward every export; fail safe.
- **Graphics-API hooks.** Hook the swapchain `Present`/`ResizeBuffers` and the device/command
  queue for **D3D11/D3D12** (older engines: the D3D9/8-era equivalents). This is where you get
  the device, back-buffer, and a per-frame callback.
- **VR runtime abstraction.** OpenXR (and legacy OpenVR) init, HMD pose sampling, per-eye
  swapchain/texture creation, and **submission to the compositor** each frame. Let the runtime
  own distortion, chromatic correction, and reprojection timing.
- **Stereo submission pipeline.** Take the engine's frame and present two eyes' worth to the
  runtime; manage the per-eye render targets.
- **Overlay + config.** An ImGui overlay and a config system for live-tunable settings
  (world scale, IPD, HUD scale, comfort toggles).
- **Utilities.** Pattern scanning, memory read/write, profiling.

## What is NOT reusable (the engine-specific half)

None of the above knows where *your* engine keeps its camera. That is always bespoke:

- locating the world→view and projection matrices (or the constant buffer that carries them);
- the engine's **coordinate basis and handedness** (getting this wrong makes the world "swim");
- distinguishing the **main scene camera** from shadow/reflection cameras;
- the engine's **frame-timing signals** (see [techniques/frame-timing](../techniques/));
- how the HUD/UI is drawn and how input is delivered.

The public frameworks express this boundary as a single interface you implement per engine — see
the [porting model](../porting/) (`IEngineAdapter`) and the [case studies](../case-studies/).

## The VR math you can lift wholesale

Independent of engine: projection-matrix decomposition, per-eye **FOV/IPD**, **world scale**,
decoupled yaw/pitch, roomscale origin, and HUD-to-depth placement. The
[techniques](../techniques/) section covers the parts that interact with engine specifics
(basis/handedness, TAA under AFR); the rest is standard VR math that every runtime SDK documents.

## Key takeaway

When you start a new mod, **do not rebuild the core** — study how UEVR / REFramework / vrframework
already did it and reuse that architecture. Spend your effort entirely on the engine-specific
half, because that is the only part that is actually new for your game.
