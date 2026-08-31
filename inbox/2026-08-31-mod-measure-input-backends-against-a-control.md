# Never trust an input API's return value — measure it against a no-input control

**From:** modding session (`doom-2016-vr`), 2026-08-31
**Kind:** engine-agnostic technique + a reusable in-process trick
**Confidence:** the two failures are `[verified-live]` history; the DOOM implementation is
`[built-not-proven 2026-08-31]`

## The problem this solves

"The input API returned success" and "the game reacted" are different facts. This portfolio has
paid for the difference twice, in two different engines:

- **XIII (2003, UE2)** `[verified-live 2026-08-28, n=1]` — 600 px of injected `SendInput` mouse
  motion produced **0.0°** of yaw, while keyboard input worked in the same session. The game takes
  the mouse via **DirectInput in exclusive mode**. Psychonauts hit the identical wall.
- **RE Village (RE Engine)** `[verified-live 2026-08-24]` — the game ignores `SendInput`
  **entirely**, even after a real struct-layout bug was fixed and the calls began reporting
  success. It does respond to `PostMessage(WM_KEYDOWN/UP)`.

In both cases the call site looked fine. The API said yes. Nothing moved.

## The technique

Do not ask the input API whether it worked. Ask **the renderer**.

1. Take a cheap, repeatable measurement of something the camera drives — a view-matrix candidate
   set, a screen-capture near-black fraction, a telemetry yaw reading. Anything that moves when
   the camera moves.
2. **Run a no-input control first.** Measure, wait exactly as long as an injection run takes,
   measure again.
3. Then run each backend for the same duration and score it **against the control**, not against
   zero.

**The control is the whole point.** Cameras drift on their own — idle sway, weapon bob, TAA
jitter, breathing animations. Without a control, "something changed" reads as success and you will
believe a dead backend works. With one, a backend only counts if it beats the floor by a clear
margin.

This is the same failure shape as XIII's wrapped-yaw incident (2026-08-28), where the *analysis*
was the bug rather than the mechanism, and the same shape as the claim-hygiene rule about a fix
that removes the symptom and the test coverage together.

## The reusable trick: stop pushing input from outside

Where a proxy/injected DLL is already in the process, there is a better route than any external
injection API:

> Post the game a `WM_INPUT` carrying a **sentinel handle**, then hook `GetRawInputData` and
> answer *that handle* with data you fabricated. The game never asks the OS about it — it asks
> you.

That sidesteps both walls above at once: **focus** stops mattering (a posted message needs none)
and **exclusive-mode capture** stops mattering (you are not going through the input stack at all).

Two implementation notes worth carrying:

- **Hook by IAT patching, not an inline trampoline.** It writes a data page rather than code, so
  it needs no disassembler and does not argue with **Control Flow Guard** — which is enabled on
  plenty of modern targets (both DOOM 2016 executables, for one).
- **The known failure mode is `GetRawInputBuffer`.** A game that reads input in bulk never calls
  `GetRawInputData`, so the hook installs cleanly and produces nothing. Log whether that import is
  present, so the first live run distinguishes "hook did not land" from "hook landed, wrong
  function".

## Suggested home in the library

A technique page under the input/automation area, and a row in the engines index for the
per-engine input route (UE2-era = exclusive DirectInput, dead to `SendInput`; RE Engine =
`PostMessage` yes / `SendInput` no; id Tech 6 = Raw Input, **untested**). That table would have
saved real time twice.

Implementation for reference (ours, MIT-spirit, write your own): `staging/doom-2016-vr/
proxy-vulkan/src/autoinput.c`, commit `0d314b1`.
