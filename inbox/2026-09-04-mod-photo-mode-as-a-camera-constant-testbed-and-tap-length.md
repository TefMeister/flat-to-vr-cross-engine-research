# Two engine-agnostic method notes from mad-max-vr, 2026-09-04

Author: modding lane (`/lm`, home PC). Create-only inbox drop for `/sr` to curate.

## 1. A shipped photo mode is a free-camera testbed for the camera constant — check whether it writes the same slot

On Mad Max (Apex engine, D3D11) the pause-menu Capture Mode is a free camera, and **it writes
exactly the same shared constant-buffer slots as gameplay** — the camera-position slot and the
per-pass clip matrix `[verified-live 2026-09-04, n=1 game]`. That turns a photo mode into the
cheapest possible rig for per-eye experiments: the scene is frozen, the camera is still and
keyboard-steerable, the HUD is gone, and a rewrite of the constant can be judged from a
screenshot pair with no timing pressure. Method: fire the per-write dump in gameplay, enter the
photo mode, move the camera a known way, dump again — if the same slots move by the expected
vector, the photo mode is a valid testbed for that engine. If a game has a photo mode, test this
before building any camera control of your own.

## 2. Synthetic key taps have a minimum length, and it is per machine

The same harness (`SendInput` scancodes) that drove Mad Max's menus with 70 ms taps on the dev
PC at 784×561 was **silently ignored** by the same game on the home PC at 1920×1080 — two `Esc`
taps, screen unchanged, game rendering and focused — while 250–300 ms holds registered every time
across ~20 presses `[verified-live 2026-09-04]`. Same family as the "arrows need
`KEYEVENTF_EXTENDEDKEY`" trap: nothing errors, it just reads as "this game ignores the keyboard".
Rule: when a key does nothing, lengthen the hold to ~0.3 s before concluding the binding is
wrong; record the working hold length in the game's control profile, per machine.
