# Criterion's Burnout engine

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** Criterion Games' in-house engine as shipped in Burnout Paradise (distinct from
  Criterion Software's licensed RenderWare middleware, which has [its own page](./renderware.md)).
- **Render API:** see the dossier for the measured specifics of the Remastered PC build.
- **Known public VR path:** none turnkey.

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| Burnout Paradise Remastered — resumed 2026-09-16; runs with our proxy 2026-09-17 | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/burnout-paradise-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`burnout-paradise-vr`](https://github.com/TefMeister/burnout-paradise-vr) |

## Shared findings

*Seeded 2026-08-26; grown by the research sweep as cross-project truths emerge.*

### A passive graphics proxy loads under Denuvo `[verified-live 2026-09-17, n=3 launches]`

A full-export **32-bit `d3d11.dll`** proxy beside `BurnoutPR.exe` loaded under Denuvo Anti-Tamper plus
the EA app, resolved 51 of 51 real exports, logged `D3D11CreateDeviceAndSwapChain`, and the game reached
its main menu. Denuvo hides the executable's own code from reading (and probably from a debugger); it
did **not** stand between the game and the graphics runtime, which is where a proxy sits. The
licensing layer was left untouched throughout.

### Windowed mode lives in a config file

`%LOCALAPPDATA%\Criterion Games\Burnout Paradise Remastered\config.ini`, `[Display]`:
`Width=1280`, `Height=720`, `WindowMode=1` `[verified-live 2026-09-17]`. See the cross-engine rule in
[techniques](../techniques/README.md#windowed-mode-for-unattended-runs-config-file-registry-or-command-line--never-the-in-game-menu).

Source: [`burnout-paradise-vr`](https://github.com/TefMeister/burnout-paradise-vr), `modding-notes/2026-09-17-first-live-look.md`.

## See also

- [engines index](../engines-index.md) — the "Bespoke / older custom engines" row.
