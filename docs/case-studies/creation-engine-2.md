# Case study: Bethesda Creation Engine 2 (Starfield) — `starfield2vr`

**Public project:** [`mutars/starfield2vr`](https://github.com/mutars/starfield2vr) by
**mutars (Sergii Permiakov)**, MIT, derived from praydog/REFramework. Study notes only — read the
repo for the authoritative source. Full credit in [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).

Targets Starfield on **Creation Engine 2** (D3D12, x64). Runs on OpenXR or OpenVR; needs
**ViGEmBus** for controller emulation; supports quad-view (Pimax/Virtual Desktop). It is the
public reference for the **Reflex-marker timing** model and for **keeping TAA and fixing it per
eye**. Contrast throughout with the [Anvil case study](./anvil-per-eye-camera.md).

## Repo shape (single-title adapter)

- `src/CreationEngine/` — the adapter: `CreationEngineEntry`, `CreationEngineCameraManager`,
  `CreationEngineRendererModule`, `CreationEngineInputManager`, `GameSettingsComponent`,
  `memory/offsets.h`.
- `experimental/` — D3D12 device/command-list interceptors, engine hooks, Nvidia-interposer and
  AA experiments.
- `sdk-lite/` — RE-style headers (`CameraHandle`, `CameraViewHandle`, `CreationRenderer[Private]`),
  leaning on **CommonLibSF** for Starfield's reverse-engineered structures.

## Frame timing — the Reflex-marker model

Creation Engine 2 already instruments its loop with **NVIDIA Reflex markers**, so the adapter
hooks the marker function and **decodes markers (0–6) into a frame timeline**:

- engine-start markers → `on_wait_rendering(...)` + `on_begin_rendering()` + `update_hmd_state()`;
- a render-phase marker → set the render frame count;
- a present-phase marker → set the presenter frame count.

Reusing the engine's own instrumentation as a reliable clock is elegant *when the engine provides
it* — the opposite situation from Anvil's [two-hook approach](./anvil-per-eye-camera.md).

## Submission — AFR with per-eye history

The renderer alternates eyes by frame parity and keeps **four past-frame buffer slots**, swapping
current/past resources via `CopyResource`. Per-eye view/projection are applied where the engine
updates its camera constant buffer; a **per-eye projection history map** enables temporal
reprojection.

## TAA strategy — keep it, fix per eye

Rather than disabling TAA (Anvil's route), Creation Engine 2's adapter **keeps TAA and repairs it
per eye**: a TAA pass swaps the temporal-history resources between alternating frames (by frame
parity) so each eye retains its own history. The result is that **TAA, DLSS, and CAS all work** in
VR — the higher-effort, higher-quality answer to the
[TAA-under-AFR problem](../techniques/#temporal-effects-under-afr). (Frame Generation is
incompatible and must stay off.)

## Required flat-game settings (tells you the constraints)

Windowed mode; disable Frame Generation, VSync, Motion Blur, Depth of Field, and Dynamic
Resolution — all effects that assume a single flat 2D frame or interpolate across frames, exactly
what AFR + per-eye rendering breaks.

## What to copy for a new engine

- Look for **existing engine instrumentation** (Reflex, a frame-index global, ETW) before
  hand-hooking frame-begin functions — it can hand you a free, robust timeline.
- The **per-eye temporal-history swap** pattern if your engine's TAA quality is worth preserving
  and its history buffers are reachable.
- The use of a **reverse-engineered SDK library** (here CommonLibSF) to get typed access to
  engine structs instead of raw offsets, where the community has built one.

## Sources

- **starfield2vr** (mutars / Sergii Permiakov) — [github.com/mutars/starfield2vr](https://github.com/mutars/starfield2vr)
- Upstream **REFramework** (praydog) — [github.com/praydog/REFramework](https://github.com/praydog/REFramework)
- Core framework **vrframework** (Elliott Tate) — [github.com/elliotttate/vrframework](https://github.com/elliotttate/vrframework)
- **CommonLibSF** (Starfield Reverse Engineering) — [github.com/Starfield-Reverse-Engineering/CommonLibSF](https://github.com/Starfield-Reverse-Engineering/CommonLibSF)
- **ViGEmBus** (Nefarius) — [github.com/nefarius/ViGEmBus](https://github.com/nefarius/ViGEmBus)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
