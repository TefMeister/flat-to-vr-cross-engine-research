# flat-to-vr-cross-engine-research

**A universal, engine-agnostic library of publicly-available knowledge for modding flat
(non-VR) games into VR.** Any modding project — for any game, on any engine — can read this to
find techniques, patterns, and worked examples it might reuse.

This repository is a **curated index and set of study notes for information that is already
public**. It is **not** a mod, it ships no game files, and it claims ownership of **none** of
the techniques or code it describes.

Two kinds of material live here, and they are always labeled: **study notes on other people's
public work** (the bulk of it — described in our own words, linked to the primary source, credited
in [`ATTRIBUTION.md`](./ATTRIBUTION.md)), and **our own first-party research**, published openly in
our project repos and contributed back here so it isn't stranded in one game's notes. The second
kind is ours to give; the first never is.

---

## The one rule that governs everything here

> **All techniques, code, and tools described in this library belong to their original
> authors, under their own licenses.** This repo describes mechanisms in our own words,
> summarizes public documentation, and **links to the primary sources.** Every idea traces back
> to whoever published it. See [`ATTRIBUTION.md`](./ATTRIBUTION.md).
>
> **We learn the *techniques*, not the code.** We do **not** copy, host, or reuse anyone's
> actual source code or files — not Luke Ross's (R.E.A.L.), not mutars' (starfield2vr /
> anvilengine2vr), not anyone's — **even when their license (MIT, BSD, etc.) would allow it,
> and even now that some of that work is free to download.** Free to download is not the same as
> ours to take. What we take is the publicly-explained idea; the implementation stays theirs.
>
> **Corrections & removals — no questions asked.** If you are credited or referenced here and
> want your name or information changed or removed, just email **td3kxlvr@proton.me** and tell us
> **exactly what to remove**, and we will remove it, no problem. You do not need to be a lawyer or
> prove anything — if it's yours and you want it gone, that's enough.

We built this because the knowledge on bringing flat games into VR is scattered across GitHub
repos, docs sites, forum posts, and write-ups. Our goal is simple: **make flat→VR modding more
widely available to everyone and gather as much public information about it as we can**, in one
readable, well-attributed place — with links back to every source, so it helps everyone and takes
credit from no one.

---

## How to use this library

1. Identify your game's **engine** and **render API** — start at
   [`docs/engines-index.md`](./docs/engines-index.md).
2. Skim the [tool landscape](./docs/landscape/) to see whether a turnkey solution already
   exists for your engine (e.g. UEVR for Unreal 4/5, REFramework VR for RE Engine).
3. If you must build it yourself, read the [engine-agnostic core](./docs/engine-agnostic-core/)
   (the reusable half) and the [porting model + checklist](./docs/porting/).
4. Cross-reference the [techniques](./docs/techniques/) (frame timing, AER, TAA-under-AFR, HUD,
   basis/handedness) and the [case studies](./docs/case-studies/) — mostly engines people have
   already solved in public, plus one (id Tech 6) documenting the *reconnaissance* stage on an
   engine nobody has converted yet.

## Structure

| Path | Contents |
|------|----------|
| [`docs/landscape/`](./docs/landscape/) | Taxonomy of public flat→VR tools and what each is for |
| [`docs/engine-agnostic-core/`](./docs/engine-agnostic-core/) | The reusable layers: D3D hooks, OpenXR/OpenVR, stereo submission, projection math |
| [`docs/porting/`](./docs/porting/) | The per-engine adapter model + a 10-milestone porting checklist |
| [`docs/techniques/`](./docs/techniques/) | Deep dives: frame timing, AER, TAA under AFR, HUD in VR, basis/handedness |
| [`docs/case-studies/`](./docs/case-studies/) | Worked public examples: RE Engine, Creation Engine 2, Anvil, Unreal 4/5 — plus id Tech 6, a recon-stage study of an engine nobody has converted yet |
| [`docs/per-game-native-mods/`](./docs/per-game-native-mods/) | The route for modern AAA engines with no injector: per-game mods & AER (Luke Ross R.E.A.L.) — informational |
| [`docs/generic-drivers/`](./docs/generic-drivers/) | The off-the-shelf route when you're not writing an adapter: vorpX & geo-11 for D3D9/D3D11, Vk3DVision for Vulkan |
| [`docs/source-available/`](./docs/source-available/) | VR source ports & SDK mods (Team Beef, Quake VR, HL2VR, GZ3Doom…) — the route when engine source is public |
| [`docs/unity/`](./docs/unity/) | Unity games: UUVR, VRGIN, BepInEx plugins — the managed-runtime shortcut |
| [`docs/runtime-layers/`](./docs/runtime-layers/) | OpenComposite, OpenXR Toolkit, VRto3D, Depth3D — the plumbing between mod, runtime & display |
| [`docs/engines-index.md`](./docs/engines-index.md) | Quick lookup: engine → render API → known VR path |
| [`docs/watch-list.md`](./docs/watch-list.md) | Where new information appears — the sources checked on a standing research cadence |
| [`ATTRIBUTION.md`](./ATTRIBUTION.md) | Every person, project, tool, and source, with links and licenses |
| [`CONTRIBUTING.md`](./CONTRIBUTING.md) | How to add — public info only, credit everyone, link primary sources |

## Contributing

Additions are welcome from anyone. The bar is simple and non-negotiable: **only publicly
available information, every claim linked to its primary source, and every author credited.**
See [`CONTRIBUTING.md`](./CONTRIBUTING.md).

## Related repositories

- **[flat-to-vr-RE-toolkit](https://github.com/TefMeister/flat-to-vr-RE-toolkit)** — the
  companion repo: the reusable **tools, Claude Code skills, setup, and engine-agnostic
  PLAYBOOK** for reverse-engineering any flat game into VR. Where this library is the *public
  knowledge* (engines, techniques, case studies), the toolkit is the *method and tooling* you
  work with. The two are meant to be used together.

## License & attribution

- The **original writing** in this repository (our summaries and notes) is licensed
  **[CC-BY-4.0](./LICENSE)** — reuse it freely, just credit this repository. The same spirit
  covers everything else we make (our tooling and our mods): **free to use with credit.**
- **We copy no third-party code or files into this repo** — we describe public techniques in our
  own words and link to the primary sources, which stay under their authors' own licenses
  (MIT, BSD, etc.) and are credited in [`ATTRIBUTION.md`](./ATTRIBUTION.md). CC-BY-4.0 governs only
  the text we wrote, never the material we point to.

## Legal & scope

Non-commercial, educational, community knowledge-sharing. Nothing here redistributes original
game assets or copyrighted engine source. The reverse-engineering techniques cataloged
(DLL proxying, hooking, injection, memory patching) are documented as they appear in public
projects; using them to mod a game requires owning a legitimate copy of that game and is subject
to that game's terms. This library takes no position beyond pointing at what is already public.
