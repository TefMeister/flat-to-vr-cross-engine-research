# RenderWare's view offset moves the eye as well as the picture

From: `/gr` estate sweep 2026-10-26 (manhunt-2003-vr). Full topic: `manhunt-2003-vr/external-research/topics/`
(2026-10-26, RenderWare view offset), dossier pointer `manhunt-2003-vr/engine-research/inbox/2026-10-26-gr-renderware-view-offset-also-moves-the-eye.md`.

aap's librw (an open reimplementation of RenderWare; `src/camera.cpp`, read online) shows `RwCameraSetViewOffset`
does two things: it shifts the picture AND moves the eye by the same amount, with the view window held at one unit.
So it is an off-axis eye shift, not a lens-only shift `[reported]`.

Why it transfers: any RenderWare game (GTA III / Vice City / San Andreas, Manhunt, Bully and others) where a VR mod
uses the view offset for per-eye lens centres will also widen or narrow the eye gap by the offset. With a headset
whose lens centres are off-centre (Quest 3: about ±0.18), each eye moves about 0.18 units; a symmetric simulator
lens has centre 0 and hides it. Check: log the x translation term of each eye's projection; non-zero means the eye
moved. Fix: move the camera frame back by the offset, or put the lens shift into the projection with no view offset.

Suggested home: the RenderWare engine page.
