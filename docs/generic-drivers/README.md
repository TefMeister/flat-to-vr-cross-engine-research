# Generic stereo drivers: vorpX, geo-11 & Vk3DVision

When a game is too old, too niche, or too closed to justify writing a full
[engine adapter](../porting/), off-the-shelf routes can still get it into stereoscopic 3D —
and, with vorpX, into a head-tracked VR view — **without touching the engine at all**. Most of this
page covers the large back-catalog of **Direct3D 9 (and older)** titles, where this approach is most
useful; the [Vulkan section](#vulkan--vk3dvision) covers the one modern-API equivalent.

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

### Vireio Perception — the free, open-source alternative
**Vireio Perception** (cybereality and contributors, **LGPL-3.0**, public since 2013) is a free
generic VR driver aimed at the same problem as vorpX: reconstructing a stereoscopic/head-tracked
view over an unmodified D3D game without engine work. It predates vorpX's commercial polish and
per-game profile library, and is less actively maintained, but being fully open source it's worth
trying first on a budget, or as a reference for how a generic driver's injection/reconstruction
layer is actually built (vorpX's internals are closed).
<https://github.com/cybereality/Perception>

---

## geo-11 — the DX11 stereo driver (D3D9 needs a wrapper first)

**geo-11** (ThreeDeeJay, Helix/3D-fix community) is a stereoscopic-3D driver that is effectively
*"a replacement for NVIDIA 3D Vision, for DirectX 11."* It produces excellent per-eye stereo and
pairs with [3Dmigoto](https://github.com/bo3b/3Dmigoto) for per-game shader fixes — but **it is
D3D11-only and does not natively support D3D9.**

> **What it is doing under the hood is documented, and it is not a black box.** The 3D Vision lineage
> this replaces worked by appending a **clip-space footer** to every vertex shader and issuing each
> draw twice — which is also why both it and geo-11 need per-game shader fixes, and why deferred and
> post-processed renderers give them trouble. The mechanism, its costs, and the static check that
> tells a driver-stereo title from a natively stereo one are written up in
> [the clip-space stereo footer](../techniques/#the-clip-space-stereo-footer-geometry-stereo-without-ever-finding-the-camera).
> It is worth reading before routing a game through any driver here — **the same technique is
> implementable by our own proxy**, with no NVIDIA driver or GPU involved.

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

## ⚠️ 3D Vision itself is discontinued — what that means for a game's native stereo toggle

`[reported 2026-09-03]` Every driver on this page descends from, replaces, or routes around
**NVIDIA 3D Vision**, so its status matters to any game that shipped a native "Stereo3D" option:

- NVIDIA announced the end of 3D Vision driver support on **2019-04-11**; **425.31 is the last
  driver that includes it**. (Wikipedia, citing NVIDIA's support-plan notice; NVIDIA's own page
  refuses automated fetches.)
- **DX11** stereo lingered to **452.06** through community workarounds and was removed with the
  RTX 30-series launch driver (456.38, October 2020). **DX9 games are *reported* to remain
  compatible on current drivers** — one source (3D Fix Manager's page), unconfirmed in this estate,
  and the most consequential line here for the many D3D9 titles being converted.
- The driver-made stereo also needs a display: a 120 Hz 3D Vision monitor with the IR emitter, a
  3DTV Play set, or **anaglyph glasses in "3D Vision Discover" mode on any GeForce**. HelixVision
  used Discover mode precisely to avoid the hardware.
- Getting 425.31 onto a current Windows generally means removing the DCH driver and installing
  offline, or a community driver-modding tool (3D Fix Manager's Drivers tab, the "3D Vision Driver
  Changer"). None of that belongs on a machine that also runs a current VR runtime.

**For a flat-to-VR project the practical reading is:** a game's own stereo toggle will usually do
nothing on a current driver, so do not spend a launch discovering that; **the value of a native 3D
Vision integration is the shader plumbing it left behind** — the stereo-parameters texture, the
enabled flag, the compiled-in clip-space shear — which a proxy can feed itself, with no NVIDIA
driver, no 3D display and no headset-versus-monitor confusion. And **none of this is a VR route**:
3D Vision drives a display, not an HMD. The mechanism is written up in
[the clip-space stereo footer](../techniques/README.md#the-clip-space-stereo-footer-geometry-stereo-without-ever-finding-the-camera)
and its takeover in
[taking over the stereo parameters texture](../techniques/README.md#-taking-over-the-stereo-parameters-texture--the-cost-above-turned-into-a-lever).

Sources: [Wikipedia, Nvidia 3D Vision](https://en.wikipedia.org/wiki/Nvidia_3D_Vision);
[Helix Mod: 3D Fix Manager](https://helixmod.blogspot.com/2017/05/3d-fix-manager.html) (Pauldusler);
[Helix Mod: Announcing geo-11](https://helixmod.blogspot.com/2022/06/announcing-new-geo-11-3d-driver.html);
[HelixVision driver notes](https://steamcommunity.com/app/1127310/discussions/0/1635291505036080879/)
(Bo3b). Researched by a `/gr` pass for `alice-madness-returns-vr`, 2026-09-03.

---

## Vulkan — Vk3DVision

The two drivers above cover D3D9 and D3D11. For **Vulkan** titles the equivalent is **Vk3DVision**
by **Helifax** (Octavian Vasilov) — a dedicated Vulkan stereoscopic-3D driver, and the spiritual
successor to his earlier **OGL3DVision** (an OpenGL wrapper for the now-dead NVIDIA 3D Vision
ecosystem). It advertises output to VR headsets, 3D Vision, and side-by-side / top-bottom /
interleaved 3D-TV formats, and was maintained through a per-game fix list.

> ### ⚠️ FROZEN: the public repository was archived by its owner on 2026-03-05
>
> `[reported 2026-09-05]` — re-confirmed firsthand this date from the GitHub repository API
> (`archived: true`, last push **2026-03-05**), after the archive was first noticed on 2026-09-01.
> The per-game fix list's newest per-title stamps are **2025-08-30** and its page footer reads
> *2020-2025*. **Read everything below as prior art on a finished project, not as a live option to
> recommend or to expect fixes from.** A Vulkan title with no fix on that list today is unlikely to
> get one.
>
> **This matters beyond one tool.** With Vk3DVision archived, the geo-11 GitHub mirror dormant since
> 2024 (its real distribution is off-GitHub, so that is a weak signal) and Vireio Perception
> untouched since 2022, **there is no evidence of an actively maintained generic stereo driver for
> Vulkan today.** The fallback layer that exists for D3D9 and D3D11 has no visible working equivalent
> on Vulkan. For an id Tech 6 or similar Vulkan target, that removes *"there is a generic driver if
> the bespoke route fails"* from the plan.
>
> **One loose end, recorded rather than resolved:** the repository's GitHub licence metadata reports
> **BSD 3-Clause**, which sits oddly against the closed-source description below. The repo hosts
> compiled releases, so the licence plausibly covers something other than the driver itself. Not
> settled here, and it changes nothing in practice — this library links and learns rather than
> reading anyone's implementation either way.

**Why it earns a place here:** it is public evidence that **per-eye override at the Vulkan level
works on real, closed, modern commercial games** — including a maintained DOOM (2016) fix and a more
developed DOOM Eternal one. When you're deciding whether a Vulkan target is even tractable, a
working third-party fix on that exact title is a useful sanity check independent of anything you
build.

**Limits, stated honestly:**

- **Closed source.** The public GitHub repo hosts compiled releases only; the project is
  Patreon-funded. So it is **feasibility proof and prior art, not something to study line-by-line** —
  which happens to match this library's own [link-and-learn rule](../../CONTRIBUTING.md) naturally,
  since there is no implementation to read even if one wanted to.
- **Stereo is not the same as 6DoF.** Its "VR"/"FullVR" naming should not be taken at face value as
  positional head tracking. Community discussion of VR options for DOOM (2016) names Vk3DVision and
  a ReShade/Depth3D route as the two known choices and reads **neither** as delivering true
  positional tracking. We have not been able to confirm the head-tracking question either way from
  public sources — treat it as genuinely open rather than settled in either direction.
- It only applies when the game is actually running its **Vulkan** renderer, which for some titles
  (id Tech 6 among them) is a separate executable or a config switch.

Links: [Vk3DVision-Public (releases)](https://github.com/helifax/Vk3DVision-Public) ·
[per-game fix list](https://3dsurroundgaming.com/Vk3DVisionGames.html) ·
[creator's Patreon](https://www.patreon.com/vk3dvision).

**The depth-reprojection fallback.** For a cheaper, lower-fidelity alternative that works on almost
anything with a readable depth buffer, **Depth3D / SuperDepth3D** (BlueSkyDefender, on ReShade)
reprojects the single rendered image using its depth buffer instead of rendering a real second eye.
Different technique, different quality ceiling — covered under
[runtime layers](../runtime-layers/).

**A caution about generic drivers on modern engines:** vorpX's Geometry-3D mode is reported by its
own users as no longer working on DOOM (2016), despite having worked at some point. Generic-driver
support on modern engines is patch-fragile — verify it works on your current build rather than
trusting a support list.

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
- **Vireio Perception** (cybereality) — [github.com/cybereality/Perception](https://github.com/cybereality/Perception)
- **geo-11** (ThreeDeeJay / Helix) — [github.com/ThreeDeeJay/geo-11](https://github.com/ThreeDeeJay/geo-11) ·
  [helixmod.blogspot.com](https://helixmod.blogspot.com/)
- **3Dmigoto** — [github.com/bo3b/3Dmigoto](https://github.com/bo3b/3Dmigoto)
- **dgVoodoo2** (dege-diosg) — [github.com/dege-diosg/dgVoodoo2](https://github.com/dege-diosg/dgVoodoo2)
- **Vk3DVision** (Helifax / Octavian Vasilov) — closed-source, releases only —
  [github.com/helifax/Vk3DVision-Public](https://github.com/helifax/Vk3DVision-Public) ·
  [fix list](https://3dsurroundgaming.com/Vk3DVisionGames.html) ·
  [Patreon](https://www.patreon.com/vk3dvision)
- **Depth3D / SuperDepth3D** (BlueSkyDefender) — [github.com/BlueSkyDefender/Depth3D](https://github.com/BlueSkyDefender/Depth3D)
- Third-party guide — [CompoundVR: vorpX](https://compoundvr.com/articles/vorpx-injection-driver-guide/)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
