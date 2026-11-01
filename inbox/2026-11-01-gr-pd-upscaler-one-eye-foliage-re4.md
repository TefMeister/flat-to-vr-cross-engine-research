# One more of the pd-upscaler one-eye family: foliage missing in the right eye (RE4, #1143)

From: `/gr` estate sweep, 2026-11-01. Adds to `2026-10-29-mod-single-frame-multipass-eye-mismatch.md` (same inbox);
does not replace it.

- praydog/REFramework #1143 "RE4 pd_upscaler one-eye foliage" (oneup03, 2024-10-31, open, no replies): foliage missing
  in the right eye in RE4 on the pd-upscaler branch, plain REFramework (VRto3D, OpenVR and OpenXR) `[reported]`. The
  VRto3D wiki lists it for the pd-upscaler build only, not the nightly `[reported]`.
- Seen on our home PC in RE4 with Talemann's RE4 VR (a custom build on that branch), with DLSS on and off
  `[reported 2026-10-31]`.
- Our static read of the branch (`visceral-re4-vr/modding-notes/2026-11-01-one-eye-counter-for-both-eyes.md`): under
  Single Frame Multipass both eyes' matrices come from one shared counter set per scene layer `[inferred-static]`; a
  candidate mechanism only, and #1339 (shadows) shows some of the family survives every stereo mode.
- Library page: docs/engines/re-engine.md, beside the SFM cost list. Topic:
  `visceral-re4-vr/external-research/topics/2026-11-01-one-eye-foliage-is-a-known-pd-upscaler-bug.md`.
