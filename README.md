# flat-to-vr-cross-engine-research

**A universal, engine-agnostic library of publicly-available knowledge for modding flat
(non-VR) games into VR.** Any modding project — for any game, on any engine — can read this to
find techniques, patterns, and worked examples it might reuse.

This repository is a **curated index and set of study notes for information that is already
public**. It is **not** a mod, it ships no game files, and it claims ownership of **none** of
the techniques or code it describes.

---

## The one rule that governs everything here

> **All techniques, code, and tools described in this library belong to their original
> authors, under their own licenses.** This repo describes mechanisms, summarizes public
> documentation, and **links to the primary sources** — it quotes only short excerpts and never
> reproduces whole source files from other people's projects. Every idea traces back to whoever
> published it. See [`ATTRIBUTION.md`](./ATTRIBUTION.md).
>
> If you should be credited and aren't, or you are a rights holder who wants a correction or
> removal, email **td3kxlvr@proton.me** and it will be fixed as soon as possible.

We built this because the knowledge on bringing flat games into VR is scattered across GitHub
repos, docs sites, forum posts, and Patreon write-ups. Gathering the public pieces into one
readable, well-attributed place — with links back to every source — helps everyone and takes
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
   basis/handedness) and the [case studies](./docs/case-studies/) of engines people have
   already solved in public.

## Structure

| Path | Contents |
|------|----------|
| [`docs/landscape/`](./docs/landscape/) | Taxonomy of public flat→VR tools and what each is for |
| [`docs/engine-agnostic-core/`](./docs/engine-agnostic-core/) | The reusable layers: D3D hooks, OpenXR/OpenVR, stereo submission, projection math |
| [`docs/porting/`](./docs/porting/) | The per-engine adapter model + a 10-milestone porting checklist |
| [`docs/techniques/`](./docs/techniques/) | Deep dives: frame timing, AER, TAA under AFR, HUD in VR, basis/handedness |
| [`docs/case-studies/`](./docs/case-studies/) | Worked public examples: RE Engine, Creation Engine 2, Anvil, Unreal 4/5 |
| [`docs/engines-index.md`](./docs/engines-index.md) | Quick lookup: engine → render API → known VR path |
| [`ATTRIBUTION.md`](./ATTRIBUTION.md) | Every person, project, tool, and source, with links and licenses |
| [`CONTRIBUTING.md`](./CONTRIBUTING.md) | How to add — public info only, credit everyone, link primary sources |

## Contributing

Additions are welcome from anyone. The bar is simple and non-negotiable: **only publicly
available information, every claim linked to its primary source, and every author credited.**
See [`CONTRIBUTING.md`](./CONTRIBUTING.md).

## License & attribution

- The **original writing** in this repository (our summaries and notes) is licensed
  **[CC-BY-4.0](./LICENSE)** — reuse it freely, just credit this repository.
- **Quoted excerpts and any referenced code remain under their upstream authors' own licenses**
  (MIT, etc.) and are credited in [`ATTRIBUTION.md`](./ATTRIBUTION.md). CC-BY-4.0 governs only
  the text we wrote, never the third-party material we point to.

## Legal & scope

Non-commercial, educational, community knowledge-sharing. Nothing here redistributes original
game assets or copyrighted engine source. The reverse-engineering techniques cataloged
(DLL proxying, hooking, injection, memory patching) are documented as they appear in public
projects; using them to mod a game requires owning a legitimate copy of that game and is subject
to that game's terms. This library takes no position beyond pointing at what is already public.
