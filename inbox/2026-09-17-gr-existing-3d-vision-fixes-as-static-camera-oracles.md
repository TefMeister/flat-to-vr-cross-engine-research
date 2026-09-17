# Existing HeliX / 3DMigoto fixes are a free static pointer to a game's camera shaders

**From:** `/gr` estate sweep, home PC, 2026-09-17.

## The finding

Across one sweep, **7 of 14 newly researched projects already have a public 3D Vision fix or official
3D path**: Dead Space 2 (official + HeliX `ShaderOverride`), Borderlands GOTY Enhanced (DJ-RK, 3DMigoto
1.3.15, D3D11), Far Cry 3: Blood Dragon (HeliX, DX9 only), Prototype (HeliX, HUD to depth), Deus Ex:
Mankind Divided (3D Fix Manager `start3d.exe`, "DirectX 12 3D"), Tomb Raider 2013 (official 3D Vision)
and Death Stranding DC (RealVR) `[reported 2026-09-17]`.

## Why it is engine-agnostic

A HeliX (D3D9) or 3DMigoto (D3D11) fix is a list of **shader hashes it had to override** and settings
naming **which constants it reads** for the projection. Before any logging proxy runs, that list says:
- which shaders reconstruct world position (and will break under a per-eye shift);
- which effects are screen-space (HUD, markers, halos) and need per-eye handling;
- often, which constant buffer / register carries the projection the fix adjusts `[hypothesis]`.

It is a static, zero-launch oracle for the camera search every flat-to-VR project starts with. The
generic-drivers page already covers geo-11 + 3DMigoto as a *driver*; this is their use as *evidence*.

Only read the fix files; copy nothing (the estate's no-copied-code rule applies to shader fixes too).

## Per-project topics

`external-research/topics/2026-09-17-*` in dead-space-2-vr, borderlands-goty-vr, far-cry-3-blood-dragon-vr,
prototype-vr, deus-ex-mankind-divided-vr, tomb-raider-2013-vr, death-stranding-vr.
