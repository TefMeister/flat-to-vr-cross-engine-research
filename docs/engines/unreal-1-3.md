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

*Seeded 2026-08-26; grown by the research sweep as cross-project truths emerge. The per-project
dossiers linked above remain the source of truth for game-specific detail.*

### Driving the game from an injected hook (UE2, from XIII)

`[verified-live 2026-08-28, n=2 — two faults, two different call sites]`
**A global `UGameEngine::Exec`-style entry point can be unsafe to call from an injected hook
regardless of which phase you call it from.** In XIII (2003) it faulted from a camera hook inside
`UGameEngine::Draw` *and* — after the render-path explanation was assumed and the tier re-armed —
from `ULevel::Tick`, with no render path in the stack. Prefer **narrowly-scoped dispatch objects**
instead: `UObject::ScriptConsoleExec` on the player controller reaches only that object's own exec
functions, and the cheats live on a separate `UCheatManager` the console reaches by hopping from
the controller. Locate it by **exported-vtable identity** (`??_7UCheatManager@@6B@`) rather than a
hardcoded offset, and re-validate on use — it is destroyed on level change.

Corollary worth checking on any UE1–3 title: **"it is a standard UE command, therefore it is
here" does not hold.** XIII's build has `God`/`Fly`/`Ghost`/`Walk`/`HealMe` but no `Teleport`,
`SetSpeed`, `Slomo`, `Invisible` or `Loaded`. Probe, don't assume.

### Input is alias-based, and useful aliases often ship unbound

`[verified-live 2026-08-28]` UE1–3 route input through named aliases (`Axis aBaseX Speed=...`,
`button bUp`) defined in the ini and bound to keys separately. A game can therefore **define an
action it never binds** — XIII ships `TurnLeft`/`TurnRight`/`FastTurnL`/`FastTurnR` with no key on
any of them, which reads as "the engine cannot do this" when in fact only the binding is missing.
Binding a spare key needs **no code change at all**. Check the alias table before building
anything to synthesise an input.

Two traps that cost real time on XIII, both plausibly family-wide:

- **The ini the game writes is not always the ini you should edit.** XIII *deletes* `User.ini` on
  exit and regenerates it at launch from `DefUser.ini`, so edits to the former always vanish and
  the template is the only durable target. Check which file survives a restart before concluding
  a binding "did not take".
- **A rotator read from telemetry may be unwrapped.** XIII's yaw is a raw accumulating integer
  that runs straight through the 65536 boundary. Applying the usual shortest-arc wrap to it turns
  a real −199° turn into an apparent +161°, which looks exactly like an input reversing direction.

## See also

- [engines index](../engines-index.md) — the "Unreal Engine 2 / 3" row.
- [OldUnreal](https://github.com/OldUnreal) — community custodians of UE1; their 227k patch is
  the foundation of the Unreal Gold project.
