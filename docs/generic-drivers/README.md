# Generic drivers for older D3D9 games: vorpX & geo-11

When a game is too old, too niche, or too closed to justify writing a full
[engine adapter](../porting/), two off-the-shelf routes can still get it into stereoscopic 3D —
and, with vorpX, into a head-tracked VR view — **without touching the engine at all**. This is
the pragmatic path for the large back-catalog of **Direct3D 9 (and older)** titles.

These are *generic drivers*, not engine-native mods: they reconstruct a VR/3D view over an
unmodified game. Expect good seated 3D and (with vorpX) head-look, but **not** true engine 6DoF,
motion controllers, or roomscale. All tools credited in
[`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).

---

## vorpX — the direct route for D3D9

**vorpX** (Ralf Ostertag) is a commercial VR 3D-driver that injects into an unmodified game. It
covers **250+ DirectX 9–12 games** (plus some OpenGL) and ships hundreds of per-game profiles,
so old D3D9 titles are squarely in its wheelhouse. Two things make it the first thing to try on a
D3D9 game:

### Its two 3D-reconstruction modes
- **Geometry 3D** — renders the scene **twice**, once per eye, for *true* stereoscopic depth
  (objects have real volume, distances read naturally). Best image, but costs ~50% framerate —
  and, importantly, **Geometry 3D only works with games that can run in DirectX 9 mode.** That
  makes older D3D9 games the *ideal* Geometry-3D candidates, exactly where the best mode applies.
- **Z-Buffer 3D** — takes a single rendered frame and uses the depth buffer to fake eye
  separation. Less natural depth, but keeps framerate high (which matters for comfortable head
  tracking). This is the fast default that works across the widest set of games.

### Its VR display modes
- **FullVR** — immersive, head-tracked view for first-person games (the closest to "VR").
- **Immersive Screen** — a large curved screen for third-person games.
- **Cinema** — a flat virtual screen for strategy/sports/menus.

### When to reach for it
A D3D9 first-person game with no existing VR mod, where you want a playable head-tracked result
quickly and don't need hands or roomscale. Try **Geometry 3D + FullVR** first for depth; fall
back to **Z-Buffer 3D** if framerate/comfort suffers. vorpX is paid software.

Docs: [features](https://www.vorpx.com/features/) ·
[Geometry vs Z-Buffer / head-tracking](https://www.vorpx.com/more-headtracking-z-buffer-vs-geometry-3d/) ·
[support FAQ](https://www.vorpx.com/support-faq/) · a good third-party overview:
[CompoundVR's vorpX guide](https://compoundvr.com/articles/vorpx-injection-driver-guide/).

---

## geo-11 — the DX11 stereo driver (D3D9 needs a wrapper first)

**geo-11** (ThreeDeeJay, Helix/3D-fix community) is a stereoscopic-3D driver that is effectively
*"a replacement for NVIDIA 3D Vision, for DirectX 11."* It produces excellent per-eye stereo and
pairs with [3Dmigoto](https://github.com/bo3b/3Dmigoto) for per-game shader fixes — but **it is
D3D11-only and does not natively support D3D9.**

To use it on a genuine D3D9 game you insert a translation layer:

```
D3D9 game  ──►  dgVoodoo2 (wraps D3D9 → D3D11)  ──►  geo-11 (+ 3Dmigoto fixes)  ──►  stereo 3D / VR
```

- **[dgVoodoo2](https://github.com/dege-diosg/dgVoodoo2)** (dege-diosg) intercepts the old
  DirectX (Glide/DX1–9) calls and re-issues them on **D3D11/12**. Once the game is presenting
  through D3D11, geo-11 can see it.
- Community reports put roughly **half of D3D9 games** as salvageable into 3D this way — some
  games wrap cleanly, others break. It is more finicky than vorpX and often needs a per-game
  3Dmigoto fix, but it can yield sharper true-stereo results and taps the whole existing library
  of DX11 Helix/3Dmigoto fixes.

Worked example from the Helix community:
[Tron 2.0 geo-11 3D fix via dgVoodoo2 (DX9→DX11)](https://helixmod.blogspot.com/2022/08/tron-20-geo-11-3d-fix-dx9-dx11-dgvoodoo2.html).
Releases: [github.com/ThreeDeeJay/geo-11/releases](https://github.com/ThreeDeeJay/geo-11/releases).

---

## Choosing between them for a D3D9 game

| Want… | Use |
|-------|-----|
| Fastest path to a **head-tracked VR view**, minimal fuss | **vorpX** (Geometry 3D + FullVR; Z-Buffer if slow) |
| **Sharpest true stereo**, willing to wrap + tinker | **dgVoodoo2 → geo-11 (+ 3Dmigoto)** |
| A game that **won't wrap** to DX11 cleanly | **vorpX** (native D3D9) |
| A game that already has a **DX11 geo-11/3Dmigoto fix** or wraps cleanly | **geo-11** route |

## Honest limits (both)

- These are **seated / head-look** experiences. No engine-native 6DoF, no motion controllers, no
  roomscale — for that you need an [engine adapter](../porting/).
- Depth quality depends on the reconstruction (Geometry > Z-buffer; geo-11 true stereo > vorpX
  Z-buffer) and on per-game quirks (HUD depth, transparent effects, shadows).
- Anything **older than D3D9** (D3D8/7, Glide) generally needs **dgVoodoo2** to reach either tool
  at all — see [`../engines-index.md`](../engines-index.md).

## Sources

- **vorpX** (Ralf Ostertag) — [vorpx.com](https://www.vorpx.com/) ·
  [features](https://www.vorpx.com/features/) ·
  [Geometry vs Z-Buffer](https://www.vorpx.com/more-headtracking-z-buffer-vs-geometry-3d/)
- **geo-11** (ThreeDeeJay / Helix) — [github.com/ThreeDeeJay/geo-11](https://github.com/ThreeDeeJay/geo-11) ·
  [helixmod.blogspot.com](https://helixmod.blogspot.com/)
- **3Dmigoto** — [github.com/bo3b/3Dmigoto](https://github.com/bo3b/3Dmigoto)
- **dgVoodoo2** (dege-diosg) — [github.com/dege-diosg/dgVoodoo2](https://github.com/dege-diosg/dgVoodoo2)
- Third-party guide — [CompoundVR: vorpX](https://compoundvr.com/articles/vorpx-injection-driver-guide/)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
