# `SendInput` silently sends nothing when the `INPUT` struct size is wrong — and that reads as "the game ignores synthetic input"

Date: 2026-09-10
Author: `/lm` modding session, `manhunt-2003-vr`, dev PC
Scope: **engine-agnostic.** Nothing here is about Manhunt, RenderWare or D3D8. It applies to any
automation harness that drives any game with `SendInput` from a scripting host.

---

## The trap

`INPUT` is **28 bytes in a 32-bit process and 40 bytes in a 64-bit one** — a `DWORD type`, then
four bytes of alignment padding, then a 32-byte union (`MOUSEINPUT` is the largest member: two
`LONG`s, three `DWORD`s, padding, and a `ULONG_PTR`).

If a harness declares it at the wrong size — very easy in P/Invoke, where the size is written by
hand as `[StructLayout(LayoutKind.Explicit, Size=28)]` — then:

```
SendInput returns 0
GetLastError()   = 87   (ERROR_INVALID_PARAMETER)
```

**A return of 0 means zero events were inserted.** Nothing is typed, nothing is clicked, and
**nothing is printed** unless the caller checks the return value. Most harnesses do not: they call
`SendInput`, sleep, take a screenshot, see no change, and record *"this game ignores synthetic
keyboard input"*.

The 64-bit host is what makes this common. PowerShell, Python and Node are usually 64-bit today,
while the sample code people copy from is often written for a 32-bit process, so the mismatch is
the default outcome rather than an unusual mistake.

## Why it costs so much

The false conclusion is **plausible and self-consistent**, so it survives:

- Games really do ignore `SendInput` sometimes — DirectInput and Raw Input both bypass the window
  message queue — so "this game ignores it" is a *believable* result, not an absurd one.
- It is reproducible: it fails identically every time.
- It sends the project down the expensive path. On the project this was found on, it motivated
  building an in-process DirectInput hook to inject state directly — real work, weeks of it,
  justified by a measurement that never happened.

On that project **every `SendInput` result recorded across six weeks had to be withdrawn at once**.
Not one of them was a test.

## The fix, and the rule

```
n = SendInput(1, ref evt, Marshal.SizeOf(typeof(INPUT)));
if (n != 1) throw / log GetLastError();
```

Two parts, and the second matters more than the first:

1. **Compute the size — never hardcode it.** `Marshal.SizeOf` / `ctypes.sizeof` gets it right for
   the host's bitness automatically. If you must use `LayoutKind.Explicit`, put the union at
   `FieldOffset(8)` and let the runtime size the struct.
2. **Assert the return value equals the number of events you passed, every call.** This is the
   general rule and it is the one worth carrying: *a silently failing input API and a genuinely
   ignored input API are indistinguishable from outside, and only one of them is a fact about the
   game.* Any input route whose failure mode is "nothing happens" needs a success signal that is
   independent of watching the screen.

## The same shape, elsewhere

This is the second time the same *class* of defect has produced a false negative on this project
inside one day. The other: an in-process hook guarded its device-vtable patch with a single global
flag, so only the **first** device created was ever instrumented — and the two devices did not
share a vtable. The uninstrumented device reported zero reads, which was recorded as *"the game
never reads the keyboard"*.

Both are the same failure: **an instrument that is not measuring looks exactly like a subject that
is not responding.** Worth a standing entry in the library's method pages —

- Before recording a negative input result, prove the channel can produce a **positive** one.
  Point the same harness at Notepad, or at any window that visibly responds, and confirm the
  keystroke lands. It costs one minute and it is the difference between a finding and a folk tale.
- Prefer instrumenting **both ends**: the sender's return value *and* a counter on the receiving
  side. On this project the in-process hook's own per-device counters are what finally separated
  "not read" from "not hooked".

`[verified-numerically 2026-09-10]` for the return value and error code; `[verified-live
2026-09-10, n=6]` that the identical calls succeed once the size is correct.
