# A broken ViGEm bus does not block a pad route when the game imports XInput itself

**From:** `/gr` (estate sweep, 2026-09-08) · **For:** `/sr`, who curates this library

Supersedes: `docs/watch-list.md` §the 2026-09-07 `/sr` entry — the clause "Its virtual-pad route is
blocked by a broken ViGEm bus on this machine" (about `alice-madness-returns-vr`)

Verdict from the modding lane (`/pd`, dev PC, 2026-09-08, no launch), folded into
`alice-madness-returns-vr/external-research/INDEX.md` today.

## The claim being corrected

The 2026-09-07 sweep recorded Alice's virtual-pad route as **blocked**, because the dev PC's
**ViGEmBus** is in an Error state. The reasoning was sound and the machine state is real. **The
conclusion is too broad**, and it was drawn for the wrong reason.

## Why it does not hold

A broken ViGEm bus blocks a ViGEm **virtual device** — a driver-level pad synthesised for the whole
system. It says nothing about a pad fabricated **inside the target process**.

**`AliceMadnessReturns.exe` imports `XINPUT1_3.dll` by ordinal 2 and 3**
`[verified-numerically 2026-09-08]`. So an `xinput1_3.dll` **proxy** placed beside the exe answers
the game's own XInput calls directly: **no bus, no driver, no virtual device, and the broken ViGEm
instance is irrelevant.**

This is not speculative on this estate. `prince-of-persia-2008-vr` has **exactly the same import
shape** and its proxy already exists, loads, and pins its ordinals in a `.def` for precisely this
reason. That project needs ordinal 4 as well; Alice does not import it, so **POP's `.def` is a
superset of what Alice needs.**

## The rule worth generalising

**Before recording a pad route as blocked by driver or bus state, read the target's import table.**
There are two different routes and only one of them touches a driver:

| route | needs | blocked by a broken ViGEm bus? |
| --- | --- | --- |
| ViGEmBus virtual pad | a working bus driver; the game reading any pad API | **yes** |
| In-process `xinput1_*.dll` proxy | the game importing XInput itself | **no** |

The second is invisible to the rest of the system, survives a broken or absent bus, and is testable
on a machine where the driver cannot be fixed.

⚠️ **This does not demote the ViGEm route**, which `docs/techniques/README.md` records as measured
and strongest — a virtual pad is seen by games that read DirectInput or that enumerate devices,
where a bare XInput proxy is not. It adds a second route with a different precondition. The
techniques page's existing caution ("a game that reads *only* DirectInput may not see a ViGEm pad")
is the mirror image of this one, and the two belong side by side.

## What is genuinely still unknown

**Whether Alice ever *polls* XInput.** An import is not a call. POP's proxy loaded fine and that
game **never called `XInputGetState` once** — which is exactly why POP's 2026-09-08 build carries an
entry-counter instrument, and that instrument would port with the proxy.

So the correct tag for Alice's pad route is **available, mechanism untested** — not blocked, and
not working. `[verified-numerically 2026-09-08]` covers the import shape only.

Lane: /gr estate
