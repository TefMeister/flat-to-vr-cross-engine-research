# Before calibrating an injected mouse delta, look for a game flag that turns the game's own acceleration off

**From:** `/gr` (estate sweep, 2026-09-08) · **For:** `/sr`, who curates this library
**Suggested home:** the input/injection technique page — this sits directly beside the existing
`SendInput` / `GetDeviceState` material.

Engine-agnostic. Two projects hit the two halves of this on the same day, from opposite directions.

## The trap

When a mod injects mouse movement — `SendInput`, a `GetDeviceState` hook, anything — the delta
passes through **two** scalings before it becomes camera rotation:

1. the OS pointer ballistics (Windows' threshold/acceleration curve, per machine, user-settable);
2. **the game's own mouse acceleration or smoothing**, if it has any.

Both are silent. The injection reports success, the game accepts the input, and the camera simply
lands somewhere other than intended — and by an amount that varies with how fast the previous
deltas arrived. A step size calibrated under those conditions is calibrated against the curve, not
against the game, so **it does not port to another machine and may not reproduce on the same one.**

## The cheap defence, in order

1. **Check whether the game ships a flag that disables its own mouse acceleration**, before writing
   any calibration code. If one exists, this is a launch option rather than an engineering problem.
2. Only then measure the OS-side ballistics, and record them next to any step size you commit.
3. Record in the project's dossier that the calibration **is only valid with that flag set** — a
   number without its conditions is the thing that later looks reproducible and is not.

## The two observations behind this

- **`alan-wake-vr`** — `AlanWake.exe`'s own command-line option table contains **`directaiming`**,
  which Remedy's v1.03 patch notes describe as **removing all mouse acceleration** (and enabling
  `-rigidcamera` with it) `[reported 2026-09-08]`. The flag's existence is `[measured 2026-09-08]`
  from the shipped binary. The same patch also reworked the low-level mouse reading routines to
  cope with low and variable frame rates — worth knowing for anything injecting under a VR frame
  budget.
- **`alice-madness-returns-vr`** — hit the OS half the same day and had to measure the dev PC's
  pointer ballistics (thresholds **(6, 10)**, acceleration **ON**, speed 6/20)
  `[measured 2026-09-08]` precisely because an injected delta may be scaled and any step size
  calibrated there would not port. That project's import table also shows the Win32 cursor/message
  path (`ClipCursor`, `GetCursorPos`, `SetCursorPos`, no Raw Input)
  `[verified-numerically 2026-09-08]`, which is the path this scaling applies to.

**A game reading Raw Input is the case where the OS half does not apply** — that is part of why
reading the import table first (the existing `/sr` advice) pays before designing an input layer.

## Confidence

`[reported 2026-09-08]` for what the Alan Wake flag does; `[measured 2026-09-08]` for the two
binary/OS observations. **The general rule — "look for the flag first" — is `[hypothesis]`**: it is
drawn from one game that has such a flag and one that hit the trap, not from a survey. It is cheap
enough to try that it does not need to be stronger than that before being written down; it should
not be recorded as established practice until a second game's flag has actually been used.

Lane: /gr estate
