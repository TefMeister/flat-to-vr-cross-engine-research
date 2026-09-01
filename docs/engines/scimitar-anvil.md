# Ubisoft Scimitar / Anvil

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** Ubisoft's Scimitar — the pre-2009 name of what became the Anvil family — sharing
  codebase lineage with Assassin's Creed (2007).
- **Render API:** **Direct3D 9** (plain, *not* D3D9Ex), 32-bit, with the `d3dx9_39` helper.
- **Known public VR path:** none for this generation.
  [anvilengine2vr](https://github.com/mutars/anvilengine2vr) (mutars) is a public multi-title
  adapter for the much later **AnvilNext 2.0** (Direct3D 12) generation — an architectural
  reference for the family, not something that attaches here.

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| Prince of Persia (2008) | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/prince-of-persia-2008-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr) |

## Shared findings

*Seeded 2026-08-26; **first populated 2026-09-01** from the Prince of Persia (2008) dossier, so
`n=1` by construction for this generation. The AnvilNext 2.0 material stays in the
[case study](../case-studies/anvil-per-eye-camera.md) — three engine generations separate them.*

- **Scimitar is Assassin's Creed (2007)'s engine, not merely its relative.** `[reported 2026-08-25]`
  Confirmed in the binary by the literal strings `AnvilScript`, `CustomAnvilBrush` and `startanvil`,
  and corroborated by contemporary press quoting Ubisoft's producer describing "an adapted version of
  the engine developed internally for Assassin's Creed". **Practical consequence:** AC1's modding
  scene is legitimate adjacent prior art for engine and format questions — while still verifying
  against the target binary, since two years and a studio change separate the builds.
- **`.forge` is the family's archive format, and extraction tooling already exists.** `[reported
  2026-08-25]` Large packed archives (up to ~1.2 GB each). A game-specific extractor ("Elika") and a
  more generic `.forge` extractor/replacer are both public. Not needed for camera work, but it means
  asset-level work would not start from zero.
- **Direct3D 9, and specifically *not* D3D9Ex.** `[inferred-static 2026-08-25]` Static imports of
  `d3d9.dll` and `d3dx9_39.dll`; `Direct3DCreate9Ex` is **absent**. Worth carrying forward, because
  Ex versus non-Ex changes windowed flip-model and GPU-thread-priority behaviour that injection and
  presentation work later depends on.
- **A from-scratch `d3d9.dll` proxy is live-verified, first attempt.** `[verified-live 2026-08-25,
  n=1]` Only `Direct3DCreate9` needed forwarding here — but that is a per-game fact, not a family
  one; see
  [a proxy DLL must export everything the target actually imports](../techniques/#a-proxy-dll-must-export-everything-the-target-actually-imports).
- **The engine takes a rich command line, and it names its own console flag.** `[inferred-static
  2026-08-25]` The executable embeds its default command line —
  `/world:POP0WORLD /fast /shadows:on /lightmode:normal /fardist:1500 /noconsole /bink:on
  /mission:pop0_root /startupmenu:on /localbigfile`. The presence of **`/noconsole`** strongly implies
  an enabling counterpart, and `/world:` + `/mission:` are a ready-made deterministic-launch recipe.
  `DebugMenu` / `DebugMenuHandler_m` class strings confirm a real developer menu exists in the binary;
  no public unlock method was found, so this one has to be solved first-hand.
- **Cutscenes and gameplay drive the camera through different paths.** `[reported 2026-08-25]`
  Inferred from a mature third-party stereo fix shipping **separate convergence presets** for the two
  — you cannot need two presets unless there are two paths. Scope live camera investigation to check
  both early rather than assuming one covers the other.

## See also

- [engines index](../engines-index.md) — the "Ubisoft AnvilNext 2.0" row.
- [Case study: Anvil per-eye camera](../case-studies/anvil-per-eye-camera.md) — from the
  AnvilNext 2.0 adapter; the family's camera handling is the part most likely to rhyme across
  generations.
