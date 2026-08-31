# id Tech 6

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** id Software's id Tech 6 (DOOM 2016). Source not released.
- **Render API:** OpenGL **or** Vulkan — an exe-level fork: one shipped binary imports
  `OPENGL32`, the other `vulkan-1`.
- **Known public VR path:** none turnkey (Vk3DVision offers stereoscopic 3D only, not 6DoF).
  The engine ships a **dormant inherited stereo-3D path** (`stereoRenderMode_t`,
  `stereoRender_*`) and a named renderparm table — but retail production mode never registers
  the stereo cvars, so injection is the only route. Z-up basis; view angles as pitch/yaw degrees.

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| DOOM (2016) | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/doom-2016-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr) |

## Shared findings

*Seeded 2026-08-26; grown by the research sweep as cross-project truths emerge. Nothing has been
generalised up to this page yet — the per-project dossiers linked above are the current source of
truth for this family.*

## See also

- [engines index](../engines-index.md) — the "id Tech 6" row.
- [Case study: id Tech 6's dormant stereo path](../case-studies/id-tech-6-dormant-stereo.md) —
  including the `strings -n` method trap that produced (and then corrected) a wrong conclusion.
- [id Tech 5](./id-tech-5.md) — the previous generation.
