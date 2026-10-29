# Single Frame Multipass: the two eyes can differ (lighting, UI opacity, shadows)

From: modding session, 2026-10-29 (Tefa's question on the home PC: "why single frame multipass is giving me
different lighting in each eye").

- Seen by Tefa on the home PC in RE games under REFramework's Single Frame Multipass (`VR_RenderingTechnique_V2=2`):
  different lighting in each eye `[reported 2026-10-28]`. All RE games were moved to Two Frame Sequential (`=1`) on
  2026-10-28 because of it (status/visceral-re2-vr.md).
- Public reports of the same family: UI background opacity differs per eye under SFM in RE3R on pd-upscaler, AFR/TFS
  cure it (praydog/REFramework#1506) `[reported]`; shadows in one eye only on a pd-upscaler dev build, not cured by
  changing the technique, cured by the stable build (#1339) `[reported]`; praydog's stated view: SFM is the preferred
  technique (fixes TAA, lets DLSS/FSR2 run without ghosting) (#773) `[reported]`.
- Likely mechanism `[hypothesis]`: SFM draws both eyes inside one game frame, so anything the engine computes once per
  frame or carries from the previous frame (auto exposure, temporal history, volumetric fog, ambient occlusion, a
  shadow pass) is either computed from one eye and reused by the other, or run twice with the second run building on
  the first. Which effect it is per game is not established; the test is SFM on, one effect off at a time, both eyes
  compared by brightness.
- Library page this belongs on: docs/engines/re-engine.md, "Single Frame Multipass is the rendering choice to pair
  with DLSS", as its known cost.
