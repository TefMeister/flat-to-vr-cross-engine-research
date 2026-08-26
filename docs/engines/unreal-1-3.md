# Unreal Engine 1–3

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** Epic Games' Unreal Engine, generations 1 through 3 — including licensee layers
  built on top of it (Ninja Theory's NTEngine layer on UE3, for Enslaved).
- **Render API:** UE1 uses pluggable render-device DLLs (OldUnreal's 227k patch adds modern
  renderers and native 64-bit builds); UE2 is Direct3D 8; UE3 is Direct3D 9.
- **Known public VR path:** none turnkey. [UEVR](https://github.com/praydog/UEVR) (praydog)
  attaches only to UE 4.8–5.x and cannot be made to attach below that floor — for these
  generations it is a conceptual reference only (see the canonical playbook's appendix on what in
  UEVR is engine-agnostic and what is UE4/5-locked). Manual build.

## Our projects on this engine

| Game | Engine dossier | All project repos |
| --- | --- | --- |
| Unreal Gold (Unreal, 1998, plus its expansion) — UE1 via OldUnreal 227k | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/unreal-gold-vr-engine-research/blob/main/ENGINE-DOSSIER.md) | [`unreal-gold-vr-*`](https://github.com/TefMeister?tab=repositories&q=unreal-gold-vr) |
| XIII (2003) — UE2.x | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/XIII2003-vr-engine-research/blob/main/ENGINE-DOSSIER.md) | [`XIII2003-vr-*`](https://github.com/TefMeister?tab=repositories&q=XIII2003-vr) |
| Enslaved: Odyssey to the West (Premium Edition) — UE3 + Ninja Theory's NTEngine layer | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/enslaved-vr-engine-research/blob/main/ENGINE-DOSSIER.md) | [`enslaved-vr-*`](https://github.com/TefMeister?tab=repositories&q=enslaved-vr) |
| Alice: Madness Returns (2011) — UE3 | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/alice-madness-returns-vr-engine-research/blob/main/ENGINE-DOSSIER.md) | [`alice-madness-returns-vr-*`](https://github.com/TefMeister?tab=repositories&q=alice-madness-returns-vr) |

## Shared findings

*Seeded 2026-08-26; grown by the research sweep as cross-project truths emerge. Nothing has been
generalised up to this page yet — the per-project dossiers linked above are the current source of
truth for this family.*

## See also

- [engines index](../engines-index.md) — the "Unreal Engine 2 / 3" row.
- [OldUnreal](https://github.com/OldUnreal) — community custodians of UE1; their 227k patch is
  the foundation of the Unreal Gold project.
