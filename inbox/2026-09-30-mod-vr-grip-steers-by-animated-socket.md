# A VR two-hand grip that steers by an ANIMATED socket re-aims the gun whenever the animation moves (2026-09-30, from `/pd` on visceral-re2-vr)

Author: modding session (visceral-re2-vr), home PC. Engine-agnostic lesson; for the library, not a single game.

## The pattern, seen in two RE Engine games with one VR mod

REFramework's two-hand grip (`RE8VR.cpp` `update_hand_ik()`, shared by RE2, RE3, RE7 and RE8) steers the
held gun by the line from the right hand to the **left-hand grip socket read from the body animation every
frame**, against the real left controller. So whenever the *animation* moves that socket while the grip is
held, the gun re-aims with both real hands perfectly still.

- **Village (re-village-scope-vr, 2026-09-21):** the first shot after a take moved the muzzle ~4.8° every
  time, because the trigger pull moves the animated hands from the carry pose to the firing pose. Fix:
  freeze the socket at the take; the drawn hand still follows the live socket. Worn, 4.8° → ~1.5°
  `[verified-live 2026-09-21, n=12 shots]` (dossier §9cf/§9cg).
- **RE2 (visceral-re2-vr, 2026-09-30):** a relaxed-walk animation spliced into the aim bank carries no
  "keep the left hand on the gun" IK track (the stock aim motions all do), so the game's own IK stops pinning
  the support hand; the shot kick then moves the socket and the same steering throws the pistol aside —
  only while the left grip is held, only on the splice `[inferred-static 2026-09-30, test build waiting]`.

## The transferable rule

When a VR mod computes a pose from a **game-animated** reference (a socket, an IK joint, a bone), any change
to the animation under the mod's hands shows up as a mod fault. Two symptoms with one cause: a jump at a
state change (fire, reload, raise) and a throw after an animation edit. The checks that find it: log the
reference's movement relative to the hand that holds it, per frame, and mark shots; the fix that works is
to **freeze the reference at the take** (or drive the mod from controllers only), while letting the drawn
hand follow the live animation.

Sources: `visceral-re2-vr/modding-notes/2026-09-30-the-relaxed-walks-carry-no-left-hand-track.md`,
`re-village-scope-vr/engine-research/ENGINE-DOSSIER.md` §9cf, §9cg.
