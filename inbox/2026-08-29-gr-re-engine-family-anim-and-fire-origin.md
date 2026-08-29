# /gr drop (visceral-re2-vr): RE Engine family facts for docs/engines/re-engine.md

Three findings from a Visceral RE2 research session that look family-wide, not RE2-specific —
candidates for the "Shared findings" section of `docs/engines/re-engine.md`. All `[reported]`
(read from public source), unverified by us.

1. **Fire-origin selector exists engine-wide in praydog's VR handling:** RE Engine titles fire
   from the camera flat; REFramework's per-game code flips a shipped enum
   (`...ShellDefine.FireBulletType`, `Camera` → `AlongMuzzle` in RE2) so bullets originate at
   the weapon's `vfx_muzzle1` joint in VR. Lesson shape: look for dormant enum variants before
   building a mechanism. Source: `src/mods/FirstPerson.cpp` in praydog/REFramework.
2. **Custom animations at runtime, cross-title recipe:** `via.motion.*` is engine-level —
   register custom motlists as `via.motion.DynamicMotionBank` (MotionBankResource →
   ResourceHolder → DynamicMotionBank → Motion component), play with `layer:changeMotion`, and
   pause+disable `via.motion.MotionFsm2` while the clip runs or the game's FSM re-drives the
   layer the same frame. Manual root motion requires `set_Position` + `CharacterController:warp()`.
   Source: github.com/godlock2000-eng/ResidentEvil2_CustomAnimationFramework_NonRTX — whose
   `docs/` folder (mot/motlist format specs, actor motion systems) is a reference-quality RE
   Engine animation write-up worth linking from the family page.
3. **Speed changes without foot-sliding:** pairing motion-layer `set_Speed` with a hook that
   scales the movement driver's returned speed keeps locomotion and animation in sync
   (Junh2x/RE9-Movement-Speed-Mod, RE9; technique looks title-portable).

Full topics: `visceral-re2-vr-external-research/topics/2026-08-29-*.md`.
