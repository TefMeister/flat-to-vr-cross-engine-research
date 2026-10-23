# phunkaeg's playbook, 2026-10-21: what each eye must own when a scene is drawn twice

From: `/gr` estate sweep, 2026-10-23. Engine-agnostic, so it is yours (`/sr`). `[reported 2026-10-23, read from the commit on
GitHub; the playbook attributes it to "an unpublished port"]`

The *VR Modding Playbook* (phunkaeg) gained new sections on 2026-10-21. Two look most useful to our estate:

- **Chapter 14 `#two-pass-ownership`** (render-pass hazard atlas): breakages when the scene is drawn twice a frame,
  each with a cheap test. Draws issued once after the eye loop (tints, damage flashes, fades) land in the last eye
  only (swap the eye order: a difference that follows the order is per-frame state); a draw that drains a shared
  list or cursor breaks the second eye (give each eye its own slice and check the counts); caches keyed on anything
  but the eye; per-frame state built as a side effect of drawing (a skeleton prepared only when the body draws); fog
  measured from each eye's own origin; a table of what is shared per frame (clock, noise and jitter, simulation
  snapshots) versus owned per eye (camera, projection, depth copy, reflections, ambient-occlusion constants, which
  crashed the GPU when left on the old projection). Also `#stereo-effects-retrofit` (fog, parallax, light LOD,
  particles in an old renderer).
- **Chapter 08 `#agents-share-a-machine`**: rules for two AI agents on one PC and one game install (numbered runs
  never reused, look before launching with a random pause, a command mailbox with sequence numbers, hand-off by patch
  not by writing into the other's checkout, "no new failures" as the merge gate). Close to our own lane rules; worth
  comparing with `CONVENTIONS.md`.

Also new: ch. 07 `#engine-state-cache`, `#actor-owner-lease`; ch. 06 `#feature-verification-ladder`; ch. 02
`#rendered-fingertip-touch`; ch. 12 `#body-on-camera-clock`; ch. 03 `#hand-roles`.

Where it may land for us: the techniques page's per-eye sections ("Anything that must match between the eyes must
advance once per FRAME", "Before re-running a render pass per eye, vet every call for state it advances") are the
same family; Hard Reset, Manhunt, Mad Max and Alan Wake all draw per eye in our own code.
Source: <https://github.com/phunkaeg/vr-modding-playbook> (docs/14-render-pass-hazard-atlas.md,
docs/08-project-process.md).
