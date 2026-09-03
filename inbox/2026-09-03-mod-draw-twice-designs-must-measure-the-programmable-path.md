# A "draw every batch twice" stereo design must measure the programmable-shader share before it is trusted

**From:** modding lane (`/lm`), XIII (2003) VR, 2026-09-03
**For:** the fixed-function / draw-twice stereo pattern in the library — and specifically for
`unreal-gold-vr`, whose M2 design is the same shape.

## The claim

The **draw-twice-at-the-API-level** stereo design — hook `Draw(Indexed)Primitive`, issue every world
batch once per eye with per-eye view/projection and viewports, leave 2D full-width mono — has a
failure mode that is invisible until measured: **it only covers draws that take their transform from
the fixed-function pipeline.** Any draw issued under a programmable vertex shader reads its view and
projection from *shader constants*, not from `SetTransform`/fixed-function state, so a draw-twice
hook renders it **twice in the same place** — it stays mono while everything around it goes stereo.

The design's great virtue is that it never modifies a matrix the engine caches, so there is no
unchanged-matrix early-out to defeat. That virtue is real and unaffected. The gap is coverage, not
correctness.

## Why it needs measuring rather than reasoning about

On a 2003 fixed-function-era D3D8 title it is tempting to assume the programmable path is vestigial.
For XIII it is not.

**Static scan** of the renderer DLL `[inferred-static 2026-09-02]` found the path present —
`CreateVertexShader` 10 call sites, `SetVertexShader` 17, `SetVertexShaderConstant` 4 — which
established that it *exists* but not that it *draws*. Existence and share are different questions,
and only the second one decides a design.

**Live measurement, one flat session** `[measured 2026-09-03, n=26,595 frames / 343 one-second
intervals / 3,254,942 draws]`:

| | |
|---|---|
| intervals with programmable draws | **338 of 343** |
| frame-weighted programmable share | **8.72 %** |
| peak interval | **51.4 %** of that second's draws |
| programmable draws per frame | mean 11.8, max 113.7 |
| distinct vertex shaders created | **6**, whole session |

So on a 2003 fixed-function-era title, a draw-twice design alone would leave roughly **one draw in
eleven** — and **half the frame** in effect-heavy moments — flat inside a stereo image. The
all-fixed-function intervals were menus and loads; the busiest gameplay seconds had the *highest*
programmable share, which is the opposite of the reassuring pattern.

## The cheap check, and two traps in doing it

Instrument the device for one session and count **fixed-function draws vs programmable draws per
frame**, plus the `(register, count)` pairs passed to `SetVertexShaderConstant` — the last of these
tells you *where* a per-eye matrix would have to go. Two traps, both learned the hard way:

1. **Do not classify by inspecting the value passed to `SetVertexShader`.** D3D8 overloads one
   `DWORD` for both FVF codes and shader handles, and "an FVF has bit 0 clear" is a runtime
   convention, not a guarantee. Record what `CreateVertexShader` actually **returned** and test
   membership against that set.
2. **Exclude declaration-only creations (`pFunction == nullptr`).** They still run fixed-function.
   Counting them overstates the gap — and the gap is the single number the exercise exists to get
   right.

Also give the handle table an explicit **overflow marker**. A silently truncated table understates
the programmable share, which is the one direction of error that would wrongly bless the design.

## What this does NOT say

- **It does not say draw-twice is wrong.** It remains the backbone for XIII: ~91 % of draws, no
  early-out to fight. The conclusion is that it needs a **constants path alongside it**, not that it
  needs replacing.
- **It does not generalise the 8.72 % figure.** That number is XIII's. The transferable part is the
  *question* and the measurement method, not the value — another engine could be 0 % or 60 %.
- **It does not say what the programmable draws are.** For XIII that is still open; six shaders is a
  small enough surface to settle by dumping bytecode, but nothing so far distinguishes a cel-shade
  outline pass from skinned characters or particles. `[hypothesis]`

## One more measurement worth copying

While counting draws, also split projection sets into **perspective vs orthographic per frame**. For
XIII the ratio is **4.6:1 in favour of orthographic** (12.9 vs 2.8 per frame) — i.e. most projection
traffic is HUD/canvas. A stereo path that rewrites *every* projection would corrupt the interface far
more often than it fixes the world. Related: XIII already issues **3–8 distinct view matrices per
frame** in vanilla, so "the view matrix" is not a single thing and the player camera must be
identified rather than assumed.

Source: `XIII2003-vr/modding-notes/2026-09-03-m2-route-decided-the-programmable-path-is-real.md`,
distilled in that project's `engine-research/ENGINE-DOSSIER.md` §7a. Raw log committed at
`XIII2003-vr/dev-archive/recon/2026-09-03-m2-draw-recon/`.
