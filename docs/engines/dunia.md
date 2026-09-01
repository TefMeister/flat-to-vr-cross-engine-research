# Ubisoft Dunia

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** Ubisoft's Dunia (Far Cry 2 onwards) — a heavily forked, closed descendant of the
  original Far Cry's CryEngine.
- **Render API:** Direct3D 9 (in the Far Cry 2 generation).
- **Known public VR path:** none for true 6DoF; vorpX can provide generic seated 3D. Manual
  build. Note the family split: the *original* Far Cry (2004) runs on Crytek's CryEngine with an
  official Mod SDK, and has [farcry_vrmod](https://github.com/fholger/farcry_vrmod) (fholger) —
  a vendor-SDK route that does **not** transfer to the closed Dunia fork.

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| Far Cry 2 (2008) | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/far-cry-2-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`far-cry-2-vr`](https://github.com/TefMeister/far-cry-2-vr) |

## Shared findings

*Seeded 2026-08-26; first populated 2026-09-01 from the Far Cry 2 dossier, so `n=1` by
construction.*

- **Head tracking is composed into the per-frame view-projection without splitting it.**
  `[verified-numerically 2026-09-01, n=1 game]` (Far Cry 2; not yet headset-tested) The HMD pose is folded into the
  combined matrix directly, never decomposed into separate projection and view halves — which
  removes a whole class of reconstruction error before it can arise.
- **Derive the axis convention, do not hard-code it.** Rather than carrying a runtime-to-engine axis
  table, the camera basis is read from the matrix's own rows every frame, so the entire conversion
  reduces to one change of basis and the camera's world position is *solved* from the matrix rather
  than assumed. That is the recommended shape for this family, and it generalises well beyond it.
- **⚠️ Two composition bugs on this engine looked exactly like handedness problems**, and both were
  caught by a numerical harness rather than by reading: a position solve mixing normalised basis
  rows with raw translation terms, and a rotation composed as the *camera* rotation where the
  transform being modified is its inverse. Reaching for the handedness knob would have masked the
  second while leaving it wrong. See
  [composition bugs that masquerade as handedness](../techniques/#composition-bugs-that-masquerade-as-handedness).

## See also

- [engines index](../engines-index.md) — the "Ubisoft Dunia" and "CryEngine" rows, including the
  citable REAC 2023 talk on Dunia's shader pipeline.
