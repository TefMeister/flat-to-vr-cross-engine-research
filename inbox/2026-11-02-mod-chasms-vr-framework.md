# Chasms VR Framework (chasmlol) — what in it is useful to us

Source: https://github.com/chasmlol/chasms-vr-framework (MIT, Rust, active: pushed 2026-11-02). Read 2026-11-02 on the
home PC, README + docs/XRSIM.md + docs/design/vr-player-controller/weapons_reload_inventory.md only. `[reported]`

What it is: a separate VR process that owns the whole VR player (head, physics hands, full-body IK, guns with mags,
holsters, two-hand aim, melee, menus) and talks to a game through a small per-game "adapter" over shared memory
(protocol v17, C header provided). One adapter exists: Oblivion (32-bit dinput8.dll).

Useful to us, without adopting the framework:
1. **Design research notes** (docs/design/vr-player-controller/*.md): cited numbers from Alyx, H3VR, HL2VR, VRExpansion,
   XRI, Auto Hand, HurricaneVR, for two-hand aim, recoil, reloads, holsters, melee. A ready reference for feel decisions.
2. **Two-hand aim smoothing** (weapons doc §4): forward = primary grip → One-Euro-filtered support hand; ease in
   ~0.15 s on the second grab; fade two-hand influence by distance from the foregrip. Directly relevant to Tefa's
   2026-11-02 idea "smoothing cut 75% when two-handing a pistol" (mod-ideas resident-evil-all).
3. **Reload design**: Alyx pulls 2 shells per reach, 1 when only one slot is left, a distinct sound on the last shell —
   matches our two-shell pouch grab; the "one slot left" and "last shell sound" details are not in ours yet.
4. **xrsim**: an OpenXR runtime with no hardware, chosen per process via XR_RUNTIME_JSON (system runtime untouched),
   scripted head and both controllers, saves both eyes per frame. Overlaps our OpenXR Simulator + VR Bench; worth
   comparing for VR Bench (both-eye captures, per-process selection). `[hypothesis]` that it works with RE Engine games.
5. **Melee rules** (docs/MELEE.md) and **full-body IK with a learned prior** (crates/ik): reference for Ashes' body rig
   and any physical melee.
6. **Scripted test range** (`rustvr --sim all`): scripted player runs every gesture and checks the game reacted — a
   model for growing VR Bench into a regression suite.

Not a fit as a whole for the RE games: they render through RE Engine + REFramework, and the framework expects the
adapter to render eyes on request and hand the body to the runtime; adopting it would be a rewrite.
