# A "read-only" REFramework Lua hook on a per-frame game function broke the feature that function drives (2026-09-21)

From: modding lane, `re-village-scope-vr` (dossier §9bn / §9bo). Engine-agnostic lesson, RE Engine evidence.

**What happened.** A diagnostic Lua script hooked `app.WeaponGunCore.updateScope` purely to capture the
gun object: the pre-hook read `args[2]` and returned nothing, the post-hook returned the value
untouched. It was read-only by intent and said so in its own header. With it loaded, the VR scope
picture showed sky in every direction; the plugin's own geometry map was sane on 116 of 259 log lines
and its rifle-vs-gaze reading sat at a 54° median. With the script removed and **nothing else
changed**, the map was sane on 49 of 49 and the reading returned to 14° — matching a known-good
session (456/462, 13°) `[verified-live 2026-09-21, n=1 launch + the wearer's confirmation]`.

**The mechanism is NOT established** `[hypothesis]`: most likely the Lua hook trampoline on a per-frame
managed method interacting with REFramework VR's double submit. The effect is solid.

**Two rules worth keeping:**

1. **Never park a capture hook on a function that belongs to the feature you ship.** Capture `this` from
   something rare and unrelated (a shot, an equip), not from the hot path of the thing being rendered.
2. **Remove every probe hook the moment its question is answered.** A disproved probe left loaded is
   not dead weight — it is live code on the hot path. Here its fifteen helper files were archived and
   the script itself was left in `autorun/`.

**And a trap in the diagnosis:** the corrupted rifle-vs-gaze reading *looked like evidence about the
user* ("the rifle was pointing 54° away from where you looked"), and was first written up that way. It
was the fault's own fingerprint. When a wearer's report and a log line disagree, check whether the log
line is downstream of the fault before using it against the report.
