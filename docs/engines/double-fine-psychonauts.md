# Double Fine's Psychonauts engine

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** Double Fine Productions' bespoke in-house engine, written for Psychonauts (2005).
- **Render API:** see the dossier for the measured specifics of the PC build.
- **Known public VR path:** none turnkey. Manual build — this is the library's exemplar of the
  fully "unattempted engine" case the canonical playbook is written for.

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| Psychonauts (2005) | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/psychonauts-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`psychonauts-vr`](https://github.com/TefMeister/psychonauts-vr) |

## Shared findings

*Seeded 2026-08-26; first filled 2026-08-31. Only one project sits on this engine, so "shared" here
means the parts that held up as engine-independent and were promoted to
[techniques](../techniques/README.md); the rest stays in the dossier.*

- **The camera object carries several matrices, and only one of them is real.** Three of the four
  found near the camera are derived outputs nothing reads back — writable, persistent, and with no
  effect on the picture. The one the engine actually culls and renders from is a full 4×4 whose
  columns carry the basis (with projection scaling folded into two of them) and whose translation
  row is the origin expressed in those axes. `[verified-live 2026-08-28]` The method that found it —
  solving for the origin and checking the answer against the negated camera position — is
  engine-independent and is written up in
  [techniques](../techniques/README.md#finding-the-camera-matrix-the-engine-actually-reads).
- **Culling follows that matrix.** Rotating it 90° rendered *less* near-black than leaving it alone
  (2.58 % against 3.80 %), which is what makes the black-void problem tractable at all on this
  engine rather than being an architectural wall. `[measured 2026-08-28]`
- **A symmetric FOV widen is a mitigation with a ceiling**, measured here at roughly ×3.0 before it
  plateaus and then reverses. Full numbers and the headset-free measurement method:
  [techniques → the void behind the player](../techniques/README.md#the-void-behind-the-player).
- **The game's own free-look clamp (~87°) bounds the camera-driving route**, so the two mitigations
  above are complementary and neither is sufficient alone.
- **Menus need a real gamepad.** Synthetic keyboard input reaches gameplay but the title and credits
  screens do not consume it, so an unattended run cannot get itself from launch into a save — a
  person has to reach gameplay first. Worth knowing before designing any automated test harness for
  a game of this era. `[verified-live 2026-08-28]`

## See also

- [engines index](../engines-index.md) — the "Bespoke / older custom engines" row.
- [techniques](../techniques/README.md) — the engine-agnostic forms of the findings above.
