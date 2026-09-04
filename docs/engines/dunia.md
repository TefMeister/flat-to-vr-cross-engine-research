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

### 2026-09-04: the AER submission path is built, and its load-bearing part is parity

`[compile-verified 2026-09-04]` · `[verified-numerically 2026-09-04, 22 assertions]`, **never run.**
The bridge previously submitted one mono texture to **both** eyes while the override drew alternate
frames from alternate eyes — half the stereo discarded and each eye shown the wrong view half the time.
It now keeps a texture per eye, writes each frame into the one for the eye it was drawn with, and
submits both every frame.

- **The hazard worth knowing before writing this on any engine** is that the eye is chosen and the
  frame is captured in different places inside one `Present` hook, with the flip to the next eye
  between them — so reading the live eye state at capture time is **off by one and swaps the eyes**.
  The fix is to latch the completed frame's eye before flipping. Swapped eyes look like working stereo
  with inverted depth, not like a bug, which is what makes it worth a section of its own:
  [latch the eye with the frame](../techniques/README.md#alternate-eye-rendering-latch-the-eye-with-the-frame-or-you-silently-swap-them).
- **One shared pose for both eyes is forced here, not chosen.** OpenVR cannot express two poses in one
  frame (issue #1253, opened 2019-11-23, still open, no Valve reply); OpenXR's projection layer can,
  but SteamVR ships no 32-bit OpenXR runtime and this is a 32-bit process. Re-read 2026-09-04, with the
  detail that a partial fix was reported for one driver only:
  [OpenXR carries a pose per view](../techniques/README.md#openxr-carries-a-pose-per-view-where-openvr-collapses-to-one).
- **A test-hygiene note from the same work**, now generalised: the parity test's first version passed
  while asserting nothing, because its sample matrix was not classified as perspective and every
  comparison reduced to `0 == 0`. It now asserts non-vacuity first.

## See also

- [engines index](../engines-index.md) — the "Ubisoft Dunia" and "CryEngine" rows, including the
  citable REAC 2023 talk on Dunia's shader pipeline.
