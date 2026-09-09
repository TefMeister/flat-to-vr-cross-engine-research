# An inverse-pair check identifies matrix FIELDS but not MAJOR ORDER — and the algebra says why

**From:** the modding lane (`/pd`), `doom-2016-vr`, 2026-09-09. No game was launched.
**For:** `/sr`, wherever the library covers locating or verifying camera/projection matrices in a
running game's memory. Engine-agnostic: it is a property of perspective projections, not of id
Tech 6.

## The technique this is about

A good way to prove that some 64 bytes in memory really are the engine's projection matrix — rather
than "16 plausible floats at a computed address" — is to use a **neighbouring field as a second,
independent check**. Engines commonly store `projectionMatrix` and `inverseProjectionMatrix`
adjacently, so:

> multiply them and require the identity.

It is a strong test. Measured on random data: **0 of 200,000 random matrix pairs pass**, while a
genuine pair passes at every plausible near/far ratio — including reverse-Z with an infinite far
plane, and a 1.3-million-to-one depth range in float32. `[verified-numerically 2026-09-09]`
Multiply **both** orders and take the worse error; a coincidence that satisfies one is very unlikely
to satisfy the other.

## ⚠️ The trap: it does NOT tell you the major order

I wrote a test asserting that a **transposed** inverse — the shape a row/column-major mix-up
produces — would be rejected. **It passed**, and the reason is structural rather than a tolerance
being sloppy.

For the standard form

```
P = [[a,0,0,0],
     [0,b,0,0],
     [0,0,c,-1],
     [0,0,d, 0]]
```

the inverse's lower-right block is `[[0, 1/d], [-1, c/d]]`, and the usual mapping
`c = zf/(zn−zf)`, `d = zn·zf/(zn−zf)` gives **`d = zn·c`**.

So when the **near plane is near 1**, `1/d` is near `−1`, that block is very nearly **symmetric**,
and its transpose is almost itself. No tolerance loose enough to accept a genuine float32 inverse
can separate them.

**Measured both ways rather than argued:**

| near plane | transposed-inverse error | verdict |
| --- | --- | --- |
| `zn = 1`, `zf = 10000` | **1e-4** | indistinguishable — the transpose passes |
| `zn = 0.05`, `zf = 65536` | **19** | clearly rejected |

Exactly as `d = zn·c` predicts. `[verified-numerically 2026-09-09]`

## What to carry

- **The check identifies WHICH FIELDS you found.** That is worth a lot: it converts a computed
  address into an identification, and it is the difference between "these floats look projection-ish"
  and "these two adjacent buffers are a matrix and its inverse".
- **It says nothing about row- vs column-major, or handedness.** Read that off the numbers — which
  element carries the `−1`, and what the depth row does. A tool that prints only a verdict will
  quietly invite the wrong conclusion here, so **print the raw matrix** and say the limit in the
  output, not only in a header comment.
- **The discriminating power depends on the near plane**, which is a knob nobody chooses for this
  purpose. If a project ever *does* need the transpose distinguished this way, a scene with a near
  plane far from 1 is the one to measure in — but reading the `−1`'s position is simpler and exact.

## Why it is worth the library's space

The failure mode is quiet and flattering: the check *passes*, prints something confident, and the
convention is still unknown. Any project verifying a matrix field in memory by an algebraic identity
is exposed to the same class of problem — **an identity that the wrong answer also satisfies**. The
general form of the lesson is to ask, of any consistency check, *which wrong answers also pass?*
before quoting what it establishes.

Full write-up:
`doom-2016-vr/modding-notes/2026-09-09-read-the-engines-own-projection-instead-of-guessing-it.md`.
Host tests live in that project's `proxy-vulkan/test/rvtest.c` (116 checks, 0 failures).
