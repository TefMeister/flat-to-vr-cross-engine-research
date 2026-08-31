# id Tech 6 input route: MEASURED — it is DirectInput 8 + XInput, not Raw Input

Supersedes: docs/techniques/README.md line ~525, the per-engine input table row
`| id Tech 6 | Raw Input | untested | [hypothesis] |`

**From:** modding session (`doom-2016-vr`), 2026-08-31
**Confidence:** `[measured 2026-08-31]` — read directly from the shipped import tables of both
DOOM (2016) executables, build `20240321-104810-ginger-fuchsia`.

## The correction

That row came from my own inbox drop of 2026-08-31, where I wrote that "DOOM 2016 is Raw Input,
not the 2003-era exclusive DirectInput, so this has a real chance here." **That was an inference
from the game's age, not a measurement, and it is wrong.**

Credit where due: the curation caught it. Whoever folded that drop into the library **downgraded
it to `[hypothesis]` and marked it untested** rather than accepting my phrasing. That is exactly
what the confidence tags exist for, and it stopped a wrong claim from hardening into a fact
before anyone acted on it.

## What the import tables actually show

`llvm-objdump -p` on `DOOMx64vk.exe` and `DOOMx64.exe`:

- **Zero raw-input imports in either executable.** No `GetRawInputData`, no
  `GetRawInputBuffer`, no `RegisterRawInputDevices`.
- **`DINPUT8.dll` → `DirectInput8Create`** — DirectInput 8 is present.
- **`XINPUT1_4.dll`** (ordinals 2 and 3) — XInput linked directly.
- **Win32 key state:** `GetAsyncKeyState`, `GetKeyState`, `GetKeyboardState`,
  `MapVirtualKeyA`, `ToUnicode`, `ToAsciiEx`.
- **Cursor:** `GetCursorPos` **and** `SetCursorPos` — the classic centre-the-cursor-and-read-the-
  drift mouse-look pattern.
- A real message pump (`PeekMessageA` / `GetMessageA` / `DispatchMessageA` / `TranslateMessage`)
  and its own `SetWindowsHookExA`.

**Suggested replacement row:**

| id Tech 6 | DirectInput 8 + XInput 1.4 + Win32 key state (`GetAsyncKeyState`/`GetKeyState`/`GetKeyboardState`) + `Get`/`SetCursorPos` mouse-look | `[measured 2026-08-31]` | No raw input at all, in either exe. Measured from the import table, not inferred. |

## The generalisable lesson, which is the real value here

**Read the import table before designing the input layer.** It takes one `llvm-objdump -p` and it
tells you which API the game actually calls, which decides the entire approach. I designed and
built a whole in-process backend around posting `WM_INPUT` and answering `GetRawInputData` — for a
game that never calls it. A two-minute static check would have prevented all of it, and instead
nearly cost a live session, since the plan was to launch and let the probe discover this.

Corollary worth adding beside it: **"the game is from year N, therefore it uses API X" is not
evidence.** DOOM 2016 shipping DirectInput 8 puts it on the same input path as XIII (2003).

## Second-order consequence worth recording

Where a game imports **XInput directly**, a **ViGEmBus virtual gamepad** is likely the strongest
injection route available, not the fallback — the game sees a real controller, with movement and
look on the sticks, and DirectInput's exclusive mode never enters into it. Our own per-engine
table should probably carry an "imports XInput?" column for exactly this reason; it turns a
guess about the best backend into a lookup.

Our rebuilt backend now answers `GetAsyncKeyState`/`GetKeyState`/`GetKeyboardState` and the
`Get`/`SetCursorPos` pair directly, and instruments `DirectInput8Create`'s `CreateDevice` to log
whether DI8 carries the keyboard/mouse here or only controllers — measuring before building the
harder path. Implementation: `staging/doom-2016-vr/proxy-vulkan/src/autoinput.c`.
