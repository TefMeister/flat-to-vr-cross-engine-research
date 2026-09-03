# The same stereo formula transposes between engines — read the matrix class before porting any per-eye code

Filed by: the modding lane (`/pd`), 2026-09-03, from `alan-wake-vr` and `alice-madness-returns-vr`
on the same day. Engine-agnostic; both projects' dossiers carry the specifics.

## The trap

NVIDIA's clip-space eye offset, `x' = x + S·(w − C)`, is the workhorse formula for D3D9-era stereo
and appears in several projects here. Written in matrix language it is:

```
row0' = row0 + S·row3        then        row0'[3] -= S·C
```

That is correct **as mathematics** and dangerously incomplete **as instructions**, because "row 0"
is not a place in memory. Where it lives depends on the shader's matrix storage class, and two
projects in this estate landed on opposite sides of it *on the same day*:

| | `alan-wake-vr` | `alice-madness-returns-vr` (UE3) |
| --- | --- | --- |
| CTAB class | `D3DXPC_MATRIX_ROWS` | `D3DXPC_MATRIX_COLUMNS` |
| register *i* holds | row *i* | column *i* |
| shader form | `dp4 o.x, c0, v` | `mul r0,c1,v.y` … `mad r0,c0,v.x,r0` … |
| `clip.x` is | `dot(c0, v)` | the `.x` **lane** across all four registers |
| the edit | `c0 += S·c3` ; `c0.w -= S·C` | `c[i].x += S·c[i].w` ∀i ; `c3.x -= S·C` |

**The two implementations are transposes of each other.** Porting one to the other mixes columns,
corrupts `clip.w`, and **still renders** — which is the worst possible failure mode, because it
produces a picture that is wrong rather than an error.

## The check, and it is cheap

Both facts are readable off disk, with no debugger and no running game:

1. **The reflection block states the class outright.** D3D9 `CTAB` records a `TypeInfo` per constant
   whose first `u16` is the `D3DXPARAMETER_CLASS`: `2 = MATRIX_ROWS`, `3 = MATRIX_COLUMNS`. Most
   tooling ignores this field; it is the single most load-bearing thing in the table for stereo work.
2. **The bytecode is the confirming second read**, and it is unmistakable once you know the shapes:
   - `dp4 out.x, c0, v` (four consecutive `dp4`s) ⇒ **registers are rows**, column-vector `M·v`.
   - `mul r,c1,v.y` + `mad r,c0,v.x,r` + … (four `mul`/`mad`s accumulating a whole float4) ⇒
     **registers are columns**, row-vector `mul(v, M)`.

Do both. The metadata alone is a single read of a field that could be stale or mis-set by a
toolchain; the bytecode alone can be ambiguous in a shader that only uses part of the matrix.

## Why the metadata is not enough on its own — a worked case

Alan Wake's `g_mLocalToView` declares a **`4×4`** type but occupies only **3 registers**. That is
only consistent with the fourth row being `[0,0,0,1]` and elided, which in turn is only consistent
with the translation living in each row's `.w` — a conclusion the class field alone does not give
you, and which the `dp4` + `mov r1.w, v0.w` pattern in the bytecode settles immediately.

## Suggested for the library

A short section under the D3D9 material — "before you write a per-eye offset, determine the matrix
storage class, two ways" — with the table above and the two bytecode shapes. It generalises past
stereo: **any** code that patches a shader constant matrix in flight (FOV override, camera
detachment, projection hacks) has to answer the same question first, and the failure is always a
plausible-looking wrong picture rather than a crash.

Worth pairing with the existing note that a constant's register index need not be fixed across a
corpus (Alan Wake: the projection sits at `c0` in 2,238 shaders and `c192` in 2,084, because a
192-register skinning palette displaces it). Together they are the two ways a "just write the
matrix" plan silently breaks.

## Both claims are tested, not asserted

Each project's suite transplants the other's implementation verbatim and demonstrates it diverges —
Alice's shows `ndc.x +0.355` vs `+0.324` and `clip.w` corrupted. Mutation testing on both
(twelve mutants on one, six on the other, all caught, controls pass) confirms the suites can
actually tell the two apart, rather than passing whatever they are given.
`[verified-numerically 2026-09-03]`
