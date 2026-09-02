# The 3D Vision stereo-parameters texture is app-written, not driver-written — and its shape is fixed

Filed by: `/gr`, 2026-09-02, from `alice-madness-returns-vr`
Suggested home: `docs/techniques/README.md`, alongside the existing Automatic-vs-Direct section

## The finding

Any UE3-era (or other 2008–2013 D3D9/D3D10) title using NVIDIA's `nvstereo.h` correction pattern —
the "app-owned texture the shaders sample for per-eye separation/convergence/eye-sign" idiom this
library already documents — uses a **fixed, publicly known resource shape**:

| Constant | Value |
|---|---|
| `StereoTexWidth` × `StereoTexHeight` | **8 × 1** |
| `StereoTexFormat` | **`D3DFMT_A32B32G32R32F`** (D3D9) / `DXGI_FORMAT_R32G32B32A32_FLOAT` (D3D10/11) |
| Marker | `NVSTEREO_IMAGE_SIGNATURE` = `0x4433564E` ("NV3D"), written into the texture by the app to identify it |

`[reported 2026-09-02]`, from NVIDIA's own `nvstereo.h`.

**More useful than the numbers: the application writes this texture itself, every frame, by calling
NVAPI (`Stereo_GetSeparation` / `Stereo_GetConvergence`) — the driver does not push values into it.**
The driver's role is limited to reading the signature and reacting; population is ordinary app-side
work using ordinary D3D resource calls.

## Why this generalises

Any project on this account whose shaders reference a stereo-correction texture of this kind — the
signature is: an 8×1 four-float render target, sampled by a large fraction of the pixel shaders, with
a `.b`-or-similar channel carrying a ±1 eye sign — is describing **this exact NVIDIA mechanism**, not
a bespoke one. Two consequences worth stating once here rather than per project:

1. **A proxy never needs driver cooperation to take over this resource.** Since the app already
   writes it from values it queries itself, intercepting the app's NVAPI calls or its
   texture-update/bind calls is sufficient — there is no race against a driver thread writing the
   same memory.
2. **The exact byte layout can be predicted before any capture**, which turns "what does this
   texture look like" from a reverse-engineering question into a five-minute resource-creation task.

## Sources

- NVIDIA `nvstereo.h`, as vendored (unmodified) in the open-source `3Dmigoto` project —
  https://github.com/bo3b/3Dmigoto/blob/master/nvstereo.h
- Full write-up: `alice-madness-returns-vr/external-research/topics/2026-09-02-nvstereofixtexture-exact-format-and-who-writes-it.md`
