# UE3/D3D9 `c0`/`c4`/`c5` is now n=2, and the second game proves it a different way

Filed by: `/pd`, 2026-09-01
Targets: `docs/engines/unreal-1-3.md` §"UE3 / D3D9 — there IS a shared view-projection, and it is at `c0`"

## What this adds

The page currently tags the register map
`[inferred-static 2026-09-01, n=1 — read from a game's own shipped Engine/Shaders/*.usf]`.
The `n=1` is Enslaved. **Alice: Madness Returns independently produces the identical map**, and it
does so from a different class of evidence, which is what makes it worth raising the confidence
rather than just adding a row.

| Register | Constant | Enslaved | Alice |
|---|---|---|---|
| `c0` (×4) | `ViewProjectionMatrix` | shipped `.usf` **source** | compiled shader **reflection** |
| `c4` | `CameraPosition` | shipped `.usf` source | compiled shader reflection |
| `c5` | `PreViewTranslation` | shipped `.usf` source | compiled shader reflection |

Alice's numbers, from `AliceGame\CookedPC\RefShaderCache-PC-D3D-SM3.upk` (45,832 CTAB tables —
43,025 `ps_3_0`, 2,807 `vs_3_0`), read with `flat-to-vr-RE-toolkit/tools/d3d9-ctab.py`:

- `ViewProjectionMatrix` — `vs_3_0` `c0`, **2,431 shaders across 576 distinct layouts, no exceptions**
- `CameraPosition` — `vs_3_0` `c4`, 1,989 shaders / 473 layouts, no exceptions
- `PreViewTranslation` — `vs_3_0` `c5`, 486 shaders / 195 layouts, no exceptions

`[inferred-static 2026-09-01]` The game was not launched.

Source reading and reflection reading can't fail the same way — one is what the engine authors wrote,
the other is what the shader compiler actually emitted for shipped content — so agreement between
them is stronger than two more `.usf` reads would be. **Suggested: `n=2`, noting the two evidence
types.**

## 🪤 A trap worth adding to the page, because it nearly produced a wrong claim here

**Split CTAB tables by shader target before aggregating registers.** In Alice, the same name
`ViewProjectionMatrix` sits at `c0` in vertex shaders and at `c4` in pixel shaders (4,126 of those,
plus 4 stragglers at `c11`). Aggregated together, `c0` looks like the *minority* case and the obvious
conclusion is the wrong one. Vertex and pixel constant registers are separate spaces.

A related display trap in the tooling: `d3d9-ctab.py` prints *sampler* registers with a `c` prefix,
so `NvStereoFixTexture sampler c1` is `s1` and does not collide with `ScreenPositionScaleBias` at
float4 `c1`.

## Tool fix that came out of it (already pushed to the toolkit)

`d3d9-ctab.py` had two defects that this exercise hit:
- it parsed the CTAB `target` field but never surfaced it, so vertex and pixel tables merged;
- `find` truncated at `--limit 20` **silently** — 6,557 matching shaders were reported as ~900 with
  no warning.

Both fixed: target is now part of a table's identity and is printed, and `find` always reports
matched-vs-shown with `--limit 0` for no limit. Anyone who ran `find` before 2026-09-01 and recorded
a total from it should re-run.

## One more, possibly for a page of its own

Alice ships **NVIDIA 3D Vision stereo compiled into its shaders**: `NvStereoEnabled` at `ps_3_0` `c3`
in **28,017 pixel shaders — 65% of every pixel shader in the game** — with an `NvStereoFixTexture`
companion sampler. `[inferred-static 2026-09-01]` For the library's purposes the general point is
that **UE3-era "3D Vision Ready" titles may carry a per-eye path as shader branch logic, discoverable
statically from the shipped shader cache** — a cheap thing to check on any D3D9 UE3 game before
assuming stereo must be built from scratch. It is a driver-era path, not an OpenXR one, and it
supplies no head tracking.
