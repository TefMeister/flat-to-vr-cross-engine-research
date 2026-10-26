# /gr: REFramework VR retakes the standing origin from the head at VR start and on every runtime recenter

From: `/gr`, 2026-10-26 (estate sweep), generalised out of `visceral-re8-vr`.

**Finding** `[inferred-static 2026-10-26]`, from praydog's REFramework source: the VR runtime starts with its
"reset origin" flag set, so the standing origin becomes the headset's position (all three axes) at the first valid
poses; nothing is saved or read back. On OpenXR the same reset happens whenever the runtime reports a reference-space
change (a recenter). The menu's "Set Standing Height" takes only the height; Lua can read and write the origin
(`vrmod:get_standing_origin` / `vrmod:set_standing_origin`, documented in the community REFramework book); the C
plugin API has none.

**Why it transfers:** every REFramework VR game (RE2, RE3, RE7, Village, and the later RE Engine titles that share the
VR mod) inherits this. A project that places a body, arms or a gun relative to the player's height gets a different
result each launch depending on the wearer's pose at VR start, and after any recenter. Pinning the height from a
calibrated value through the Lua API is the generic stop-gap.

**Where it is written up:**
<https://github.com/TefMeister/visceral-re8-vr/blob/main/external-research/topics/2026-10-26-the-standing-origin-is-taken-from-the-head-at-vr-start-and-on-every-recenter.md>.
Credit praydog (REFramework) and the REFramework book's authors. Suggested home: `docs/engines/re-engine.md`, near
the two-hand grip and Single Frame Multipass notes.
