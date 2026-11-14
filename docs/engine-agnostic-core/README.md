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

## Things a flat game builds around ONE viewpoint (a pattern to watch, one engine so far)

A flat renderer may place some things relative to "the camera position" and draw some pictures only once a frame,
because with one eye there is only one of each. Drawn twice for two eyes, they come out at the wrong depth or slide
with the head. The examples so far, all from one Gamebryo VR mod's public release notes (df4l's FNVVR Reborn for
Fallout: New Vegas, v0.2.0 to v0.16.0) `[reported 2026-11-14]`:

- a **sky dome** centred on the one camera: clouds move against the head and read as a low ceiling; fixed by centring
  it on each eye;
- a **sun glare sprite** turned to the one camera: it rolls with the head;
- **camera-facing leaves** (billboards): they turn and roll with the head;
- a **water reflection** and **shadow maps** made once and shared: square shapes on the water and cut shadows in one
  eye; fixed by a reflection per eye (shadow maps stay once a frame);
- distant water turning grey and flat at the sides of the view.

What to do with it: when a far or sky element "swims" against the near world in VR while the middle distance holds,
look first for one of these single-viewpoint placements. Our Begin Again VR bridge has an open fault of that look
(the skyline sliding 2-6 px a frame with head sway, 2026-11-13) `[hypothesis]` that it is the same family. **One engine
so far:** a second engine, or our own confirmation, is needed before this is a rule.

## Key takeaway

When you start a new mod, **do not rebuild the core** — study how UEVR / REFramework / vrframework
already did it and reuse that architecture. Spend your effort entirely on the engine-specific
half, because that is the only part that is actually new for your game.

## Sources

- **vrframework** (Elliott Tate) — [github.com/elliotttate/vrframework](https://github.com/elliotttate/vrframework)
- **REFramework** (praydog) — [github.com/praydog/REFramework](https://github.com/praydog/REFramework) · [reframework.dev](https://reframework.dev/)
- **UEVR** (praydog) — [github.com/praydog/UEVR](https://github.com/praydog/UEVR) · [docs.uevr.io](https://docs.uevr.io/)
- Runtimes/libraries: [OpenVR](https://github.com/ValveSoftware/openvr) · [OpenXR SDK](https://github.com/KhronosGroup/OpenXR-SDK) · [Dear ImGui](https://github.com/ocornut/imgui) · [MinHook](https://github.com/TsudaKageyu/minhook) · [safetyhook](https://github.com/cursey/safetyhook)
- **FNVVR Reborn** (df4l) — [github.com/df4l/FNVVR/releases](https://github.com/df4l/FNVVR/releases)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
