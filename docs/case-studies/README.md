# Case studies

Study notes on engines that have been brought into VR **in public**. Each is a description of
how the public project solved the problem, with links to its source — **not** a copy of that
source. All credited in [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).

| Engine | Public project | What it demonstrates |
|--------|----------------|----------------------|
| **Unreal Engine 4.8–5.x** | **[UEVR](https://github.com/praydog/UEVR)** (praydog) | Native-stereo activation via engine reflection; three render modes |
| **Capcom RE Engine** | **[REFramework](https://github.com/praydog/REFramework)** (praydog) | Activating an engine's built-in OpenVR path; the upstream of most adapters |
| **Bethesda Creation Engine 2** | **[starfield2vr](https://github.com/mutars/starfield2vr)** (mutars) | [Reflex-marker timing; keep-and-fix-TAA per eye](./creation-engine-2.md) |
| **Ubisoft AnvilNext 2.0** | **[anvilengine2vr](https://github.com/mutars/anvilengine2vr)** (mutars) | [Two-hook timing; per-eye camera matrix; disable-TAA](./anvil-per-eye-camera.md) |

## Why case studies matter

The [porting checklist](../porting/) tells you *what* to do; the case studies show *how three
different engines solved the same problems differently* — which is the fastest way to see what is
truly engine-agnostic versus what you must re-derive. When you're stuck on a milestone, diff your
situation against the engine here that most resembles yours.

## The recurring pattern

Across all of them the shape is identical:

1. one **untouched universal core** (D3D hooks + VR runtime + stereo submission),
2. one **per-engine adapter** (find the camera, apply per-eye matrices, drive timing, handle
   HUD/input),
3. **per-game offsets** that need maintenance across patches.

The differences are concentrated in exactly two places — the **frame-timing signal** the engine
exposes, and the **temporal-effect (TAA) strategy** — plus the engine's **basis/handedness** and
its **main-camera discriminator**. Everything else is shared.
