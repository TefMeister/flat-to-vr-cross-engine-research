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

### ⭐⭐ 2026-09-10/11: the first headset run, and what it establishes about this engine

`far-cry-2-vr` was worn for the first time on 2026-09-10. **Per-eye stereo parity and head rotation both
work** (alternate-eye rendering through a `winmm` proxy and an OpenVR bridge, with equal frames submitted
per eye measured in the same run) `[verified-live 2026-09-10, n=1]`. Two defects fell out, and both are
engine-level facts rather than implementation slips:

- **⭐⭐ Dunia culls for its own camera.** With head rotation applied at the view-projection layer, the
  wearer reported *"looking behind me things don't render, there is no black void but only ground and
  sky, nothing else is showing until i turn with my mouse, then things pop into existance"*
  `[verified-live 2026-09-10, n=1]`. **Note the symptom shape: ground and sky still draw, objects do
  not** — so this engine culls per-object while drawing terrain and skybox globally, which is much
  easier to misread as streaming or LOD than the hard black wedge other engines give. The full
  cross-engine treatment, including why only two of the three available routes can cure it, is in
  `techniques/` → *"The void behind the player"*.
- **⭐⭐ The first-person weapon takes the world's separation and doubles.** At a realistic IPD the world
  fused while the weapon showed as two; lowering global separation helped without resolving it
  `[verified-live 2026-09-10, n=1]`. Standard cause — the viewmodel is drawn with its own projection
  (own FOV, much nearer near-plane), so one global figure cannot serve both. Ladder of fixes in
  `techniques/` → *"A frame has at least THREE stereo regimes"*.
- Also observed, and correct rather than broken: fixed-eye debug modes freeze one eye in a headset by
  design (only that eye is submitted, so the other holds its last frame), and **~16% of uploads failed
  the camera-position solve** while rotation worked — a per-pass subset that receives rotation without
  the positional offset.

#### ⚠️ Public prior art exists for this engine, and two items change the cost of the work

- **vorpX's DirectVR reportedly works in Far Cry 2** — a user reports it functioning but only from
  in-game "bed" saves rather than menu saves, **and that weapons are *"not in scale with the rest of
  game elements"***, plus black bands `[reported 2026-09-11, vorpX forum, 2020-07-29]`. Two consequences:
  **something already locates a Dunia camera-rotation address**, so writing rotation into the engine's
  own camera is not speculative here; and **the weapon depth problem in this game is an already-known
  symptom**, not a novel discovery.
- **HelixMod's Far Cry 2 (DX9) 3D Vision fix** (DHR, 2013-01-04) fixes the crosshair and the effect
  passes (smoke, water, dust, fire) and binds `O`/`P` convergence presets for aiming — with
  DarkStarSword later switching convergence on right-mouse-held. ⚠️ **It does not claim to fix the
  weapon model**, which independently supports "low convergence was good enough under 3D Vision" rather
  than "the weapon was never a problem".
- **⚠️ `Far Cry 2 Multi Fixer` (FoxAhead) patches Dunia in process memory at runtime** rather than
  editing files, explicitly to survive Steam's integrity checks. **Anyone patching this engine should
  read it first — it is both a precedent and a collision check.**
- Community reports place an `fFOV` desired-FOV multiplier in **`25_cameras.xml`** in Dunia game data
  `[reported 2026-09-11, unverified]` — cheap to confirm for anyone who can read the archives.

⚠️ **No public Far Cry 2 VR mod exists** beyond the vorpX profile, and **no published Dunia camera
yaw/pitch addresses or view-matrix offsets** could be found — so the memory route has an existence proof
and no published coordinates. The open-source **Far Cry 1 VR mod** (fholger) is **not transferable**: it
builds against the CryEngine Mod SDK with engine-level access.

Generalised from [`far-cry-2-vr`](https://github.com/TefMeister/far-cry-2-vr), its 2026-09-10 headset run
and its 2026-09-11 research pass.

## See also

- [engines index](../engines-index.md) — the "Ubisoft Dunia" and "CryEngine" rows, including the
  citable REAC 2023 talk on Dunia's shader pipeline.
