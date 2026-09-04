# A ViGEm virtual XInput pad drives an XInput game's movement AND look, no injection into the process (DOOM 2016, id Tech 6, 2026-09-04)

Engine-agnostic input finding from `doom-2016-vr` (write-up
`doom-2016-vr/modding-notes/2026-09-04-ringcam-scan-cap-too-small-and-virtual-pad-drives-doom.md`).
Tool already in the toolkit: `flat-to-vr-RE-toolkit/tools/virtual-pad.py` (ViGEmBus + `vgamepad`).

**The result:** on a game that imports XInput directly (DOOM 2016 does; check with
`llvm-objdump -p` for `XINPUT*`), a virtual Xbox 360 pad created by ViGEmBus is bound by the game as
a genuine controller. Pushing the left stick walked the player (~820-unit pure translation, camera
basis unchanged); the right stick turned the view (~98 deg pure yaw, position unchanged); each
reversed cleanly when pushed the other way. `[verified-live 2026-09-04, n=2 per axis with reversal]`
The game even raised a "Controller Disconnected" toast when the pad was destroyed on script exit —
positive confirmation it had bound the virtual device, not merely that Windows enumerated it.

**Why it beats synthetic keyboard/mouse for camera RE:**
- **Focus-independent.** XInput is polled by the game regardless of which window is foreground, so
  none of the SendInput "must be the foreground window" fragility applies. (SendInput look on this
  same game needs focus.)
- **Bypasses DirectInput exclusive mode.** The trap that swallowed XIII's `SendInput` yaw as 0.0 deg
  and made RE Village ignore `SendInput` outright does not arise: the pad is a different device class
  the game already listens to.
- **No in-process hook.** Nothing is injected to move the camera; the OS delivers the input through
  the API the game already calls. This is the same route ENSLAVED needs for its controller-chord
  debug camera (both thumbstick clicks), which a keyboard can never send.

**The check before you rely on it:** `python virtual-pad.py check` must show a pad appear on an
XInput slot and round-trip a stick value; and the target must import XInput (not only DINPUT/raw
input). If a game reads *only* DirectInput for the pad, a ViGEm XInput pad may not reach it — untested
across engines here, so treat "XInput imported" as the precondition, `[hypothesis]` otherwise.

**Method note that generalises:** each axis was confirmed by an ISOLATED, REVERSIBLE motion read off
the engine's own camera global (walk with basis fixed; yaw with position fixed; then undo each),
not by a luma-difference score. Reversal is the discrimination scene noise and idle drift cannot fake.
