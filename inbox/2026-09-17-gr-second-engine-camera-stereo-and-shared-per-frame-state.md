# From /gr: a public two-view stereo design that shares per-frame state between eyes

**From:** `/gr` estate sweep, 2026-09-17 (found while researching `witcher-2-vr`).

The **CyberpunkVR Port** by dariulone (MIT, https://github.com/dariulone/cyberpunk-vr-port) documents
an approach that looks engine-agnostic `[hypothesis]`:

1. **The second eye is a real second engine camera,** a render-to-texture view that runs the frame
   graph from its own position with its own projection, rather than draw-call duplication or
   alternate-eye rendering `[reported]`.
2. **Effects that must not differ between eyes are computed once per frame and shared:** sun shadow
   cascades, the shader clock, foliage wind, the reflection march. Its README names this as what stops
   blinking shadows and jittering foliage `[reported]`.
3. **Views are identified by a stable camera name hash,** not by draw order `[reported]`.

Point 2 is a transferable checklist item for any two-view stereo on the estate: when one eye
flickers against the other, look for per-frame state being advanced twice. Suggested home: the
core-patterns docs, credited to dariulone. Project-specific detail stays in
`witcher-2-vr/external-research/topics/2026-09-17-redengine-vr-prior-art-cyberpunk-port-and-witcher-3.md`.
