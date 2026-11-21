# Fallout: New Vegas and Fallout 3 now exist as matching decompilations (Gamebryo / Creation-era engine)

From: an ordinary session on the Build PC, 2026-11-21, for `it-never-gets-old-vr` (Tefa: "this pile of gold").
For the Gamebryo engine page and the case-studies / decompilations list.

- **fnv_decomp** <https://github.com/momusdigital/fnv_decomp> (CC0): FalloutNV.exe 1.4.0.525, 68,657 functions
  rebuilt as C++ that compiles to the same bytes at the same addresses; **f3_decomp**
  <https://github.com/momusdigital/f3_decomp> (CC0): Fallout3.exe 1.7.0.3, 63,681 of 63,682 matching `[reported]`.
- Why it generalises: many names come from two **Xbox 360 development PDBs** (Aug 2010), so the Gamebryo 2.6 /
  Bethesda layer (TESWaterSystem, Sky, BSShader, InterfaceManager, menus, VATS, Pip-Boy) is readable with original
  names, parameters and locals. Anyone modding Gamebryo-era Bethesda games for VR (TTWVR, FNV/FO3 VR mods) can read
  the engine instead of disassembling it; Oblivion-era code shares much of the layer.
- It fits TTW lists: the FNV4GB patch changes only the header and the `.bind` stub, so addresses hold `[measured
  2026-11-21]`. Example lookup: 0x004E3320 = `TESWaterSystem::UpdateLODWater` (far water: per reference, binds the
  reflected or the plain water shader, clears them under water) `[verified-numerically 2026-11-21, the lookup]`.
- Tool: `it-never-gets-old-vr/tools/fnv_where.py` (address -> function, source file and line; name search; print
  source).
- Credit: momusdigital and the fnvdecomp.com / f3decomp.com contributors. Starred, followed.
