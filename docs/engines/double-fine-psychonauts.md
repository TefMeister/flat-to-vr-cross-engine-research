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

### ⭐ Two visibility gates, and only one follows the camera

`[reported 2026-09-02]` The engine culls in **two stages**, which matters to every camera-moving mod
on it. A per-object frustum test (`ECamera::BoxVisible`, taking a bounding box) follows the camera
basis — that is the gate the 2026-08-28 yaw sweep measured and the one the void work moved. Underneath
it, the `.plb` level format ships a **`VisibilityTree` separate from the collision tree and the
navmesh**: an octree whose per-leaf bit-buffer is sized from `LeavesCount − 1`, i.e. one bit per other
leaf — a precomputed **from-region PVS** keyed on *where the camera is*, not where it looks.

Consequences for the projects on this engine, and they point in opposite directions:

- **First person is probably unaffected by the PVS.** Moving the eye to Raz's head is a translation
  small enough to stay inside the same leaf.
- **A flown free camera is affected.** Outside the level the current leaf's visible set can be empty,
  producing a black frame that no amount of transform work will fix.
- **The residual void that survived the 2026-08-28 mitigations may be this gate, not the frustum.**
  The distinguishing test is cheap and needs no headset: frustum culling varies *smoothly with yaw*,
  a PVS shows *no yaw dependence and a step change with position*.

The generalised form, with the diagnostic written out, is in
[the void behind the player](../techniques/README.md#the-residual-may-be-a-second-gate-a-pvs-steps-with-position-a-frustum-sweeps-with-yaw).

### The community's own tooling is a signature source for this binary

This game has an unusually strong open-source ecosystem for its age — a mod loader and API extender
(**Astralathe**, GPLv3) and a level-format toolchain (**PsychoPortal**), both by Jill
(`scrunguscrungus`), credited in [`ATTRIBUTION.md`](../../ATTRIBUTION.md). Beyond being documentation,
they publish **byte signatures** for engine functions, which can be scanned against your own
executable: several of this account's independently-identified addresses were corroborated that way
in one static pass, and the engine's own names for them recovered (confirming, for instance, that the
per-frame hook site is the top-level render entry point rather than an inner helper).

Their addresses are for their build and remain **leads until verified locally**; the signatures are
the transferable part. Method and cautions:
[a public reimplementation of your game is a signature source](../techniques/README.md#a-public-reimplementation-of-your-game-is-a-signature-source-not-just-a-reference).

## See also

- [engines index](../engines-index.md) — the "Bespoke / older custom engines" row.
- [techniques](../techniques/README.md) — the engine-agnostic forms of the findings above.
