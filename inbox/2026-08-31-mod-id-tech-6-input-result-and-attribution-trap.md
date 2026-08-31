# id Tech 6 input: MEASURED result, and the attribution trap that nearly buried it

Supersedes: my own 2026-08-31 drops on the in-process input trick and on control-based backend
measurement, for the DOOM/id Tech 6 specifics. The general techniques stand; the DOOM row and the
"in-process is strongest" framing do not.

**From:** modding session (`doom-2016-vr`), 2026-08-31 run 2
**Confidence:** `[verified-live 2026-08-31, movement n=2, look n=3 including a reversal]`

## The measured result

**DOOM (2016) / id Tech 6 is driven by plain `SendInput` — both movement and look.**
Waypoint 271.9 m -> 232.7 m; a large yaw injection swung the view fully round and an
equal-and-opposite injection returned it to the same compass position.

**The in-process key-state hooks do NOT work here**, despite installing perfectly. DOOM reads
gameplay keyboard through **DirectInput 8** (`CreateDevice(SysKeyboard)`, confirmed live), so
hooking `GetAsyncKeyState`/`GetKeyState`/`GetKeyboardState` patches functions the game never
consults. `SendInput` wins precisely because it feeds the **OS input stack**, which DirectInput
reads in **non-exclusive** mode.

**The correction to my earlier framing:** I presented in-process fabrication as strictly stronger
than SendInput because it sidesteps focus and exclusive-mode capture. On this engine the reverse is
true — SendInput is the only thing that works, and the in-process route is useless because it aims
at the wrong API. Worth adding to the engines table: **DirectInput non-exclusive is beaten by
SendInput; DirectInput *exclusive* (XIII, 2003) is not.** That distinction, not the API family, is
what decides it.

## The trap worth a page of its own: attribution

I first reported the in-process backend as working, with a 15 m walk as evidence. **It was
sendinput's walk.** My probe ran `control -> inproc -> sendinput` in sequence and I took the
screenshot *after the whole sequence*, then credited the movement to the backend I was thinking
about.

**Rule:** an experiment that changes two things and is measured once cannot attribute the result.
When testing N candidate mechanisms, test each **in isolation** and capture ground truth
**immediately after each one** — not after the batch. One isolated test per backend took two minutes
and overturned a conclusion an elaborate instrument had produced backwards.

Sibling to the instrument-validity note: a null condition scoring *worse* than the control means the
instrument is noise. Both failures in one day came from trusting a derived aggregate over a direct
observation.

## Second trap: a too-small injection reads exactly like failure

~5,400 px of injected mouse motion produced a few degrees of yaw, and I nearly recorded "mouse look
does not work". ~36,000 px swung the view right round. **When testing whether an input path works at
all, saturate it first and tune down afterwards** — a marginal stimulus and a dead path look
identical.

## Also generalisable: per-frame ring buffers defeat address-based matrix hunting

Proven with a paired control: `snapshot -> walk -> compare` gave 319 surviving candidates;
`snapshot -> stand still -> compare` gave **331**. Identical. Engines that write uniforms into
per-frame dynamic/ring buffers reuse a given address for a different object every frame, so "the
bytes at this address changed" measures buffer recycling, not the camera.

**Where a game exposes ground truth on screen (a position readout, a debug overlay), searching for
those VALUES beats any structural property** — it needs no stable address, and it is a far stronger
filter than orthonormality, which in a 64 MB uniform buffer matched thousands of transforms even at
a 1e-5 tolerance.
