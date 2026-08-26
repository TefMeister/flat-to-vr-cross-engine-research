# Capcom RE Engine

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** Capcom's RE Engine (Resident Evil 7 onwards, Devil May Cry 5, Monster Hunter
  Rise, and others).
- **Render API:** Direct3D 11 / Direct3D 12 (Vulkan on some platforms).
- **Known public VR path:** **turnkey** — the engine ships its own OpenVR path, and
  [REFramework](https://github.com/praydog/REFramework) (praydog and contributors) activates it.
  The best-supported non-Unreal case in this library. REFramework also exposes a Lua scripting
  API and a native C++ plugin API, which is where our own work in this family lives.

## Our projects on this engine

| Game | Engine dossier | All project repos |
| --- | --- | --- |
| Resident Evil Village (2021) — picture-in-picture sniper scope — native C++ REFramework plugin | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/re-village-scope-vr-engine-research/blob/main/ENGINE-DOSSIER.md) | [`re-village-scope-vr-*`](https://github.com/TefMeister?tab=repositories&q=re-village-scope-vr) |
| Resident Evil 2 Remake (2019) — Visceral | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/visceral-re2-vr-engine-research/blob/main/ENGINE-DOSSIER.md) | [`visceral-re2-vr-*`](https://github.com/TefMeister?tab=repositories&q=visceral-re2-vr) |
| Resident Evil 2 Remake (2019) — arcade controls — retired; superseded by Visceral | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/arcade-controls-re2-vr-engine-research/blob/main/ENGINE-DOSSIER.md) | [`arcade-controls-re2-vr-*`](https://github.com/TefMeister?tab=repositories&q=arcade-controls-re2-vr) |

## Shared findings

*Seeded 2026-08-26; grown by the research sweep as cross-project truths emerge. Nothing has been
generalised up to this page yet — the per-project dossiers linked above are the current source of
truth for this family.*

## See also

- [engines index](../engines-index.md) — the "Capcom RE Engine" row.
