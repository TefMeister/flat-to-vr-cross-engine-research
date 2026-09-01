# The clip-space stereo footer: how to get real geometry stereo without ever finding the camera

Filed by: `/gr` (estate sweep), 2026-09-01
For: `/sr` — suggested home: `docs/techniques/README.md`, as a new entry in **Stereo submission
strategies** and/or its own section near **Dormant native stereo paths**.
Source project: `alan-wake-vr` (where the question arose), but the technique is engine-agnostic.
Confidence: `[reported 2026-09-01]` — from NVIDIA's own published developer documentation. Nothing
here has been built or run by us.

## Why the library needs this

`docs/generic-drivers/` records **geo-11** and the HelixMod lineage as *drivers you point at a game*,
and describes 3D Vision only as *"the now-dead NVIDIA 3D Vision ecosystem"*. What is not recorded
anywhere is **the mechanism those drivers use** — which is documented in public by NVIDIA, is about
fifteen lines of thinking, and is implementable by any of our proxies without an NVIDIA driver, an
NVIDIA GPU, or 3D Vision existing at all.

It is a direct answer to the hardest recurring question in this estate: *"how do I get two eyes out of
a closed renderer whose view matrix I cannot find?"* — the exact wall that `docs/techniques/` →
**Finding the camera matrix the engine actually reads** exists to attack from the other side.

## The mechanism

NVIDIA's 3D Vision **Automatic** produced true per-eye geometry — not reprojection — like this:

1. The driver *"monitors vertex shader creation, and adds a footer to each shader."*
2. The footer works in **clip space**, chosen because it is *"directly before the perspective
   divide"*, so a horizontal shift there changes apparent stereoscopic depth **without altering the
   rasterised location or the z-buffer depth** of the resulting fragments.
3. The footer is one line:

   ```
   ClipPos.x += Separation * (ClipPos.w - Convergence)
   ```

4. *"application issued draw calls are substituted for two separate draw calls — one for the left eye
   and one for the right eye"*, `Separation` positive for one eye and negative for the other, each
   into its own eye buffer.

Read what that costs you: **nothing about the application's camera, view matrix, projection matrix,
or coordinate handedness needs to be known.** `w` is the view-space depth every projection already
produces. The two scalars are yours. This is the cheapest known route from "I can see the draw calls"
to "I have two correct eyes", and it is orthogonal to — and combinable with — the head-tracking
problem, which it does not solve.

## The costs, which are documented and real

- **Not every draw should be stereoised.** The driver used heuristics *plus a hand-built per-game
  profile* from NVIDIA's own QA. Skyboxes, HUD, full-screen quads, and anything already in screen
  space must be excluded. This is the same discrimination problem the library already documents as
  **Main-camera discrimination**, arriving from a new direction — and it is the reason 3D Vision
  needed thousands of per-title profiles and HelixMod needed per-game fixes.
- **Post-processing and deferred renderers break.** Both *unproject* from window space back to world
  space, and that unprojection has to undo a clip-space transform the shader does not know was
  applied. NVIDIA's documented remedy is to publish the live `Separation` and `Convergence` into a
  small special texture (`StereoParmsTexture`, built by the freely-published `nvstereo.h` header)
  which the **application's own shaders** sample in order to invert the transform. A mod doing this
  itself would have to patch those shaders too, or accept broken screen-space effects.
- Convergence is a comfort parameter with no on-screen representation; it needs tuning per title.

## Two things it explains retroactively

- **`docs/generic-drivers/` → geo-11** is described upstream as *"a replacement for NVIDIA 3D Vision,
  for DirectX 11"*, and 3Dmigoto per-game "fixes" are shader edits. That whole ecosystem now has a
  named mechanism in the library rather than being a black box you route a game through. Worth a
  cross-link both ways.
- **Vintage "separation is applied at the projection stage" comments.** `docs/techniques/` →
  *Dormant native stereo paths*, caution 2, warns that vintage stereo may apply separation as *"a
  projection/screen-space step"* rather than as two offset eye views, and treats that as a limitation
  to route around. This is the same family of technique, described from the engine side — which means
  the caution is right that it is not positional per-eye geometry on its own, and *also* that it is a
  known, characterised idiom of the era rather than a puzzle. (Offered as context for that section.
  **Not** a claim about any specific engine — id Tech 6's own case is separately and better resolved
  in `doom-2016-vr`, whose research shows the BFG-lineage `vieworg` genuinely *was* moved per eye.)

## A cheat sheet for whoever writes this up

| | 3D Vision **Automatic** | 3D Vision **Direct** (NVAPI) |
| --- | --- | --- |
| Who splits the draws | driver | application |
| Who owns separation/convergence | driver + per-game profile; user tunes with Ctrl+F3/F4 | application |
| What the app must do | nothing — but its post-processing breaks unless it reads `StereoParmsTexture` | render left, render right, Present |
| How selected | default | `NvAPI_Stereo_SetDriverMode(NVAPI_STEREO_DRIVER_MODE_DIRECT)`, **before device creation** |

That last row is a genuinely useful diagnostic for the whole estate: **a game whose binary references
NVAPI stereo is not necessarily a game that renders two eyes.** In Automatic mode the game's stereo
symbols are a *correction* layer over work the driver did, so finding `Separation`/`Convergence`
uniforms in a 2008–2013-era renderer is evidence of 3D-Vision-awareness, not of a native two-eye
path. Checking which driver mode the title requests separates the two, statically.

## Sources

- https://archive.docs.nvidia.com/gameworks/content/technologies/desktop/nv3dva_background.htm
- https://archive.docs.nvidia.com/gameworks/content/technologies/desktop/nv3dva_stereoscopic_issues.htm
- https://help.autodesk.com/cloudhelp/ENU/Scaleform-Help/scaleform_help/3di/stereoscopic/nvidia.html
- https://github.com/NVIDIA/nvapi/blob/main/nvapi_lite_stereo.h
- https://docs.nvidia.com/nvapi/nvapi__lite__stereo_8h.html
