# Avalanche Engine

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** Avalanche Studios' in-house Avalanche Engine (the Just Cause / Mad Max lineage).
- **Render API:** see the dossier for the measured specifics of the Mad Max build.
- **Known public VR path:** none turnkey. Manual build.

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| Mad Max (2015) | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/mad-max-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`mad-max-vr`](https://github.com/TefMeister/mad-max-vr) |

## Shared findings

*Seeded 2026-08-26; first populated 2026-09-01 from the Mad Max dossier, so `n=1` by construction.*

- **The shader bundle ships loose, with reflection intact — and Denuvo does not cover it.**
  `[inferred-static 2026-09-01]` Anti-tamper blocks attaching to the executable, which is what
  stalled this project at first injection. It does **not** protect `Shaders_F.shader_bundle`, which
  sits in the game root carrying **1363 DXBC shaders with their `RDEF` reflection chunks intact**
  (Shader Model 5 throughout) across **84 distinct constant-buffer layouts**. Camera work is
  therefore startable with no debugger, no capture and no launch. See
  [read the shipped files before you attach anything](../techniques/#read-the-shipped-files-before-you-attach-anything).
- **The per-object camera transform is named and located:** `cbuffer InstanceConsts`, 368 bytes,
  used by 112 shaders, with **`WorldViewProjMatrix` at offset 0** (64 bytes) and `SkyMaskProjMatrix`
  at +288. Other variants carry `SpotProjectionMatrix1..3`, `SpotShadowMatrix1` and
  `PointlightProjectionMatrix1` — shadow and light passes, identified as things *not* to touch. A
  few small `$Globals` buffers hold a plain `ViewProj` / `WorldViewProj` at +0 in post and effect
  shaders.
- **⚠️ The limit, which bounds the technique for this family:** the shared per-frame buffer
  `GlobalConstants` (651 shaders, 2352 bytes) has **no recoverable member names** — its RDEF type
  record declares a raw `float4 Globals[20]` array, filled from C++, rather than a struct. So
  reflection names the per-object matrix but **cannot** name a shared view matrix. If one exists it
  must be found by value, though reflection has already narrowed that to about twenty float4 slots
  in one named buffer.

## See also

- [engines index](../engines-index.md) — the "Bespoke / older custom engines" row.
