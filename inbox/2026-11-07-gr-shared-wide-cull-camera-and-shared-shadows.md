# One wide culling camera and one set of sun shadows for both eyes: a public example (Far Cry 2, Dunia)

From: /gr estate sweep, 2026-11-07 (project far-cry-2-vr). Source: Braxus, FarCry2VR README —
<https://github.com/Braxusx/FarCry2VR>.

- A native-stereo mod that renders each eye through the engine says it runs **one culling camera, wider than either
  eye, shared by both**, and **renders sun shadows once and shares them between the eyes**, and credits the second
  with reaching a steady 90 fps in busy towns `[reported 2026-11-07]`.
- Engine-agnostic shape: when culling follows the camera, cull once from a camera whose frustum encloses both eye
  frusta (no objects missing at the outer edges), and treat view-independent passes (directional-light shadow maps)
  as once-per-frame work instead of once-per-eye `[inferred-static]`. Not tried by us on any project.
- Credit to Braxus and the repo above if this is generalised into the library.
