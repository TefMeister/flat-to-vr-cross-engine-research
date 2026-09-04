# When the IK setters are dead ends, own the getter the solver reads

Filed by: modding (`/lm visceral-re2-vr`), 2026-09-05. Engine: RE Engine (RE2 remake) via a
REFramework C++ plugin, but the method is engine-agnostic.

## The pattern

A game places a character's support hand on a weapon joint. Every *setter* the IK controller
exposes was a dead end: one accepted a target and did nothing (the game rewrote it each frame), one
kind could not be enabled by either call route, one target setter threw, two solver objects were
null. The thing that worked was the opposite direction — **find the read path the solver actually
consumes, and override what it returns.**

1. **Locate the getter chain statically.** Decompiling the "update constraint" routine showed it
   read a different field (the weapon→hand attach), and the "get aid target" getter simply
   returned a joint's world matrix — so the target was weapon-side data, an anchor, and something
   player-side read it. `[inferred-static 2026-09-05]`
2. **Prove the read path live with call counts.** Post-hook the candidate getters and count calls
   per second next to the frame counter. Both getters ran at exactly one call per frame, and the
   two counts were always equal — which, with the next step, also revealed that one getter calls
   the other. `[verified-live 2026-09-05, n=1 weapon]`
3. **Override the return in the post-hook.** The getters return a `Nullable<mat4>` through a
   hidden return-buffer pointer; at return `*ret_val` is that buffer (`HasValue` byte at 0, the
   matrix at +0x10). Adding 10 cm to the translation moved the *skeleton's wrist joint* by 10 cm
   (measured joint-to-joint, no hook in the read); shifting both getters moved it 20 cm
   (additive, so nested); switching off returned it to 0.000 — under the aim state too.
4. **Measure whether the solver blends or snaps** with a 10 Hz trace across the toggle edge. It
   snapped within one sample, so any "smooth, not snap" requirement is the mod's to provide by
   blending the value it returns — trivial, since the hook returns a fresh matrix every frame.

## Why it generalises

- A solver's setters can be overwritten by the game later in the same frame, silently. A getter
  the solver reads *is* the value at the moment it matters; owning it wins the ordering fight.
- Per-frame call counts are a cheap, decisive test of "does the engine read this at all" before
  building anything on it — zero calls means the consumer reads natively and the hook must move
  down a level.
- Equal counts on two candidates plus additive effect is a two-line proof of nesting.

## Trap to record

A hook framework's post-hook may hand you `ret_val` as a pointer to the register that holds the
hidden return-buffer pointer, not to the struct itself — dereference once, then check the
`HasValue` byte before touching the payload. And a plugin's own reads of the getter go through the
same hook: useful as a self-check that the buffer edit lands, misleading if not subtracted from
the call count (here ~1/s against ~345/s).

Evidence: `visceral-re2-vr/dev-archive/recon/2026-09-05-arm-kind-and-reload/run9-aid-target-override.txt`;
write-up `visceral-re2-vr/modding-notes/2026-09-05-the-aid-joint-is-an-anchor-and-the-arm-kind-is-dead.md`.
