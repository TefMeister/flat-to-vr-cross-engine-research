# /gr: in stereo, things the engine centres on ONE camera point (sky dome, sun glare, facing leaves) and pictures it makes ONCE (water reflection) need per-eye treatment

From `/gr` for `ba-ttr-vr-bridge` (2026-11-14). Source: df4l's FNVVR Reborn, a New Vegas VR mod built into a DXVK fork,
public release notes v0.1.0 to v0.17.0 — https://github.com/df4l/FNVVR/releases. All `[reported]`; Gamebryo
(Fallout: New Vegas) only so far, so this is one engine, one project.

- v0.2.0: the sky dome centred on each eye: before, clouds moved against the head and looked like a low ceiling; the
  sun glare likewise made to face each eye instead of rolling with the head.
- v0.5.0: square shapes on the water and cut shadows in the right eye came from one water reflection picture and the
  shadow maps being shared between eyes; each eye given its own reflection, shadow maps drawn once per frame.
- v0.10.0: distant water no longer turning grey and flat at the sides of the view.
- v0.16.0: tree and bush leaves (turned to face the camera) no longer turning and rolling with the head.

The general shape: anything the flat game positions relative to "the camera position" (sky domes, sun sprites,
camera-facing foliage, view-dependent reflection planes) silently assumes one viewpoint; rendered twice it sits at
the wrong depth or slides with head motion. Our own `ba-ttr-vr-bridge` has an open fault of that look (the far
skyline sliding 2-6 px a frame with head sway while the middle distance holds still, 2026-11-13) `[hypothesis]` that it
is the same family. Bar for the library: wait for a second engine or our own confirmation before generalising.
