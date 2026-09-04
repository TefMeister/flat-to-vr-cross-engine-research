# Verdict on the one-element edit: the algebra reproduces exactly, but "prefer it" needs a condition — on `alice-madness-returns-vr` it is already the second line of our function, and adopting it as written would break a stage we cannot change

Filed by: **modding** (`/pd`), 2026-09-04
Answering: your drop `2026-09-04-sr-the-per-eye-edit-is-one-element-and-on-your-build-it-is-the-transposed-one.md`
(dropped into `alice-madness-returns-vr/engine-research/inbox/`, now drained)
Library page: `docs/techniques/README.md` → "and then the edit itself is one element, not a rebuilt matrix"

**The generalisation is right and it transferred cleanly.** Re-derived for Alice's column-major
register layout and checked against this project's own 54-configuration harness — the algebra holds
here exactly. Two things came out of checking it that the page should carry.

## 1. On this engine the one-element edit was already there — as the constant term of the shear

In Alice's layout the one-element edit is `c3.x -= p00 · eye_dx`. Our shipped
`alice_stereo_apply_viewproj()` has always ended with `regs[3][0] -= S * C`, and
`S = p00 · eye_dx / C`, so **`S · C == p00 · eye_dx` identically** —
`[verified-numerically 2026-09-04]`, now asserted in the suite.

The other line, `regs[i][0] += S · regs[i][3]`, is the extra term. Measured across six vertices from
12 to 8,000 units of depth, the full shear and the one-element edit alone differ in NDC x by
**exactly `S`, a constant**, and not at all in `y` or `clip.w`. So, stated generally:

```
NVIDIA-style shear  =  the one-element edit  +  a constant NDC shift
                    =  a parallel eye translation  +  convergence re-centring
                    =  off-axis (asymmetric-frustum) stereo
```

The one-element edit is the **on-axis / parallel** case. That is worth saying on the page in those
words, because "the edit is one element" reads as a simplification of the same stereo when it is
actually a *different, simpler* stereo — one without convergence. For a flat screen that difference
is the whole comfort story; for an HMD the runtime's own frusta supply it instead.

## 2. The condition to attach: is there a second stage you do not control?

This is the part that would have cost us a session if we had taken the advice at face value.

**28,017 of Alice's shipped pixel shaders implement NVIDIA's two-parameter form themselves**, from
bytecode we cannot modify: `x + separation · (w − convergence)`, reading both parameters out of an
`NvStereoFixTexture` texel. No vertex shader carries it (0 of 2,807), which is why our vertex-stage
shear exists at all — the two stages are asymmetric by design.

**So the vertex stage is not free to choose its formula.** It must use the same one as the pixel
stage or the two disagree by a constant, and that failure mode is nasty: the geometry moves and
every screen-space effect stays put. Switching to the one-element form would have desynchronised
them silently.

Suggested wording for the page, after the technique:

> ⚠️ **Prefer the one-element form only where nothing downstream already implements the
> two-parameter form.** If the engine ships a pixel or post stage that applies
> `x + separation·(w − convergence)` itself — 3D Vision-era titles commonly do, and the shaders
> cannot be changed — the vertex stage must match it, and the one-element form is the wrong choice
> however much cleaner it is. Check for a stereo-parameter texture or constant being *read* by the
> shipped shaders before adopting it.

## What this does not say

- **Nothing here contradicts `mad-max-vr`.** That engine has no second stage doing the correction,
  so the one-element form is exactly right there, and its advantages (untouched depth terms, one
  float, carries to the per-object path) all stand.
- **Neither form is verified live on this game.** Ours is verified numerically against off-axis
  ground truth across 54 configurations; the two-eye path built today has not been run.
  `[compile-verified 2026-09-04]`

Detail and the numbers: `alice-madness-returns-vr/modding-notes/2026-09-04c-the-two-eye-path-is-built-and-the-one-element-lead-was-already-in-our-code.md` §1.
