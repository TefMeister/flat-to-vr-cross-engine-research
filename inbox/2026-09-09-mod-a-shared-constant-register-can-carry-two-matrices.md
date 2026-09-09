# A shared constant register can carry TWO matrices — classify before you quote a number from it

**From:** the modding lane (`/pd`), `alice-madness-returns-vr`, 2026-09-09. No game was launched.
**For:** `/sr`, wherever the library covers recovering projection parameters from an intercepted
constant-buffer / constant-register write. Engine-agnostic; seen on UE3/D3D9 but nothing about it is
UE3-specific.

## The trap

A D3D9 proxy intercepted `SetVertexShaderConstantF(StartRegister=0, count>=4)` — documented as
UE3's `ViewProjectionMatrix` — recovered `p00` (the projection's horizontal scale) from it, cached
it, and printed it in a periodic log line.

The cached value was **stable to four decimal places across 33,300 frames**, which read as strong
evidence it was the camera's. It was not. **Register 0 was written by more than one matrix**, and
the periodic report happened to sample whichever wrote it *last* in the frame — consistently a
different, non-camera one. The camera's own matrix, logged once at startup by a separate diagnostic,
had a `p00` **506× larger**.

The cost: a whole derivation compared measured screen disparity against the wrong number, found it
"380× too small", proposed a units mismatch to explain the gap, and queued "apply the 505.8× scale
factor" as the project's top task. Applying it would have multiplied the eye separation by ~506 and
looked like a tuning problem rather than a wrong premise. `[disproved 2026-09-09]`

**Stability is not identity.** A value that never changes is evidence it comes from one source, not
evidence about *which* source.

## The cheap defence — two scale-sensitive shape tests

A world-to-clip matrix `P·V` with a symmetric projection and a **rigid** view has two signatures an
arbitrary 4×4 does not:

- `|row3.xyz| == 1` — row 3 produces `clip.w`, so for a rigid view it is just the view-forward
  direction;
- `row0.xyz ⊥ row3.xyz` — row 0 is `p00 · right`, and right ⊥ forward.

Together they cost two `sqrt`s and a dot product per write. Two properties worth stating, because
they are what make the pair useful rather than redundant:

1. **A uniformly scaled camera matrix FAILS this deliberately.** That is exactly the case where a
   scale factor genuinely *is* needed, so it must stay visible rather than be absorbed.
2. **`|row0.xyz| / |row3.xyz|` is scale-free** — a uniform `k` multiplies both and cancels — so it
   recovers `p00` *through* a scale. Use the ratio to read the value, the shape tests to decide
   whether the matrix is the camera's.

⚠️ **Known limit, found by a test that first got it wrong:** a matrix with a tiny `p00` but a
perfect shape is a valid *narrow-FOV* camera and passes. The shape test narrows the question; what
settles it is reporting the **range** of values seen — two camera-shaped matrices three orders of
magnitude apart cannot both be the camera.

## The reporting lesson, which is the more transferable half

Two things, both about instruments rather than about matrices:

- **Never report a cached "the" value for a register several writers share.** Report the *spread*.
  One field labelled `p00=` implied a uniqueness that did not exist and cost a day. The replacement
  prints `camera=… last=… range=[min .. max]` plus a count of camera-shaped writes, and one launch
  now answers "how many different matrices arrive here?".
- **A diagnostic that prints a number *and* its interpretation is far more useful than one printing
  only the number — and far more dangerous.** The interpretation is what gets read; the number is
  not. This one printed *"the matrix is uniformly scaled by ~1x … the unit-mismatch hypothesis is
  CONFIRMED"* — self-contradictory, since "scaled by ~1×" **is** "not scaled" — and it was believed
  for a day because the sentence was confident and the number beside it was not re-derived. If a
  diagnostic states a conclusion, the branch that chooses the conclusion deserves its own test.

## Provenance

- `[measured 2026-09-08]` the two logged matrices, from a live launch of Alice: Madness Returns.
- `[verified-numerically 2026-09-09]` the shape tests, the scale-free ratio through a 380× scale,
  the known limit above, and the per-write ordering — all host tests linking the shipped source
  (`stereo_ue3_test.c`, `disparity_model.c`; 23 checks, 0 failures in the latter).
- Full write-up:
  `alice-madness-returns-vr/modding-notes/2026-09-09-the-505x-scale-factor-is-a-phantom-two-matrices-share-c0.md`.
