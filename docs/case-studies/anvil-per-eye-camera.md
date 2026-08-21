# Case study: Ubisoft AnvilNext 2.0 (Assassin's Creed) — `anvilengine2vr`

**Public project:** [`mutars/anvilengine2vr`](https://github.com/mutars/anvilengine2vr) by
**mutars (Sergii Permiakov)**, MIT, derived from praydog/REFramework. Study notes only — read the
repo for the authoritative source. Full credit in [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).

Targets Odyssey, Valhalla, Mirage (Origins in progress) on **AnvilNext 2.0** (D3D12, x64), via a
`dxgi.dll` proxy and OpenXR. It is the public reference for the **two-hook timing** model and for
a clean **per-eye camera module**.

## Repo shape (multi-title adapter)

- `extern/vrframework/` — the untouched **Layer-1 core** (as a submodule).
- `games/<Title>/engine/` — a full **Layer-2 adapter** per game: `EngineEntry`,
  `EngineCameraModule`, `EngineRendererModule`, `EngineTwicks`, `ModConfig`, plus
  `memory/offsets.h` (**Layer 3**), and a per-game `sdk/ACVRGfxContext.h`.

This is the "one core, N per-game adapters" structure in the flesh — the titles share the adapter
*shape* and differ mainly in offsets.

## Frame timing — the two-hook model

`EngineRendererModule` installs two `safetyhook` inline hooks (no Reflex markers to lean on):

- a **begin-engine-frame** hook increments the engine frame counter and pumps the overlay;
- a **begin-render-frame** hook syncs `render_frame_count = engine_frame_count`, then calls
  `on_begin_rendering(...)` and `update_hmd_state(...)`.

Counters advance together (engine → render → present); the offsets are per-title and need
maintenance across game patches. Contrast with the Creation Engine 2
[Reflex-marker approach](./creation-engine-2.md).

## The per-eye camera module (the instructive part)

`EngineCameraModule` is a **stateless singleton** with **no pose/eye member variables** — it
reads live from the VR layer each call and mutates the engine's own `GfxContext` / `CameraNode` /
`GlobalGfxConstants`. It installs ~9 inline hooks; the two that matter most:

**View composition — `onCalcFinalView`** (gated on HMD-active and not-showing-UI). The HMD pose
is composed and **right-multiplied onto the engine's existing view matrix** with a basis
round-trip:

```
transform = Y_UP_TO_Z_UP_BASIS × rotation_offset × hmd_transform × eye × INV_UP_TO_Z_UP_BASIS
view = view × transform
```

Read right-to-left on a vector: engine **Y-up → Z-up**, apply per-eye offset, apply live HMD
pose, apply user rotation offset, convert **Z-up → Y-up** back. This is exactly the porting-guide
milestone-5 formula in production. The basis round-trip is what prevents the world from
"swimming" — the dominant first-injection bug. (Eye selection isn't here; the `eye` transform is
the current eye, chosen by AFR parity elsewhere.) A `removePitchFromZUpMatrix` path implements
**decoupled pitch** at the matrix level.

**Projection — `onCalcProjection`**: when the HMD is active **and the far plane exceeds a
threshold (~1201 units)**, the engine projection is replaced with an **asymmetric off-center
frustum** from the runtime. The far-plane threshold is the **main-camera discriminator** — it
tells the real scene camera (large far plane) apart from shadow/reflection cameras (small far
planes), so VR only hijacks the world view. Asymmetric (off-center) frustums are correct for VR.

Supporting hooks handle the forward vector (aiming), HUD viewport scaling + scissor expansion,
graphics-context copy (per-eye submission), and feeding the composed orientation back into the
engine's camera so gameplay systems see the VR view.

## TAA strategy — disable it

`EngineTwicks` **byte-patches TAA off** (and letterbox off) rather than fixing temporal history
per eye — the cheaper of the two [TAA-under-AFR strategies](../techniques/#temporal-effects-under-afr).
Dialogs remain letterboxed, a known limitation.

## What to copy for a new engine

- The **stateless per-eye camera module** design (read live VR state, write engine structs).
- The **basis round-trip** shape — but derive *your* engine's up-axis/handedness matrices; Anvil
  is Y-up, others (e.g. id Tech) are typically Z-up.
- A **cheap main-camera discriminator** — Anvil uses far-plane magnitude; render-target size also
  works.
- The **two-hook timing** model when your engine exposes no Reflex/marker instrumentation.
