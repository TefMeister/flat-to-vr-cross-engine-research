# Photo/Capture-mode FOV sliders are a free projection testbed — and their menus are often mouse-only (Mad Max, Apex engine, 2026-09-04)

Complements the home PC's drop of the same morning, `2026-09-04-mod-photo-mode-as-a-camera-constant-testbed-and-tap-length.md` (photo mode as a testbed, tap length); this one adds the FOV slider result, the aspect anchoring, the mouse-only UI and the keymap file.

Engine-agnostic points from `mad-max-vr` (write-up `modding-notes/2026-09-04b-capture-mode-fov-slider-moves-the-projection-columns.md`):

1. **A shipped photo mode with an FOV slider gives you the projection's focal columns at several known
   angles for free.** In Mad Max the slider changed only |col 0| and |col 1| of the shared view-projection
   (hfov 58°–117°), leaving the eye and forward vector alone `[measured 2026-09-04, n=6 dumps]`. That is a
   built-in reference for any per-eye projection rewrite: set a known angle, read the columns, compare.
   Photo modes also freeze the scene, which removes animation noise from A/B dumps.
2. **Which FOV is anchored shows up across two window aspects.** Same hfov (80.5°) at 16:9 and at 1.40:1,
   vfov different — so this engine anchors the horizontal and derives the vertical. One launch per aspect
   answers it for any game; it matters because a VR patch must scale the right column.
3. **Photo-mode settings tabs may ignore the keyboard entirely.** Two sessions tried E / Tab / arrows for a
   tab switch; a mouse click on the tab label worked first time, and slider values moved only via clicks on
   their `<`/`>` arrows — not bar clicks, not knob drags. When an in-game UI shows a mouse glyph in its hint
   row, drive it with absolute clicks (a `click <x> <y>` primitive is now in
   `flat-to-vr-RE-toolkit/tools/game-harness.py`) before spending keypresses.
4. **Read the game's own keymap file before guessing keys.** Mad Max's `settings.ini` stores actions as
   alphabetical indices (A=0…Z=25); it named the first-person-driving key and the enter-vehicle key before
   either was pressed. Config-backed beats community-reported, and both beat guessing.
