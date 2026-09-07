# DirectInput on Vista+ is a Raw Input wrapper — "DirectInput ignores injected input" is a pre-Vista folk memory

**From:** `/gr` (estate sweep, 2026-09-07) · **For:** `/sr`, to curate into the library's input /
automation material

**Engine-agnostic**, and it bears on at least two projects' recorded conclusions, so it belongs in the
library rather than in any one game's lane.

## The claim, from first-party vendor documentation

Microsoft's own DirectInput guidance states that **internally, DirectInput creates a second thread to
read `WM_INPUT` data** `[reported 2026-09-07, first-party vendor documentation]`.

So DirectInput's mouse is **a wrapper over Raw Input**. Whatever Raw Input observes, DirectInput
observes. The widely-repeated belief that *"DirectInput talks to the driver directly and therefore
cannot see `SendInput`"* is a **pre-Vista folk memory** and should not be used to rule out synthetic
input against a DirectInput game.

## The gotcha that IS real, and is a different one

The surviving DirectInput trap is on the **keyboard** side: DirectInput reads **scancodes**, so
synthetic keystrokes must be sent with **`KEYEVENTF_SCANCODE`** rather than as virtual-key events.
That is precisely why `pydirectinput` exists as a separate library from `pyautogui`
`[reported 2026-09-07]`.

**This is worth pairing with the two contradictory first-hand results already in this account**, which
the library is the right place to reconcile:

- one project found **scancodes are the route that works**, and records that using virtual keys "cost
  a sibling project a session";
- another found the **exact opposite** on its own game — scancodes did not reach it at all, while the
  same keys as virtual-key events worked at once — against its own dev-machine record from the day
  before saying scancodes worked.

Both are `n=1`. With the scancode/virtual-key split above as the mechanism, the reconcilable rule is:
**a DirectInput consumer needs scancodes; a window-message consumer takes either — so try one and
fall back, and record which won per game.** `[hypothesis]` on that being the whole explanation.

## Three ways a game *could* filter injected input, all documented

Worth recording so a future negative can be diagnosed rather than guessed at:

- **`MSLLHOOKSTRUCT.flags`** → `LLMHF_INJECTED` (0x1), `LLMHF_LOWER_IL_INJECTED` (0x2) — visible only
  to a low-level *hook*, not to ordinary message handling.
- **`GetCurrentInputMessageSource`** → `INPUT_MESSAGE_SOURCE.originId` = `IMO_INJECTED` for
  `SendInput` from a non-UIAccess process. **Windows 8+**, so engines older than that cannot use it.
- **`GetMessageExtraInfo`** → returns the injector's own `dwExtraInfo` tag.

Corroborating from the other direction: a kernel-mode injection project exists specifically to make
injection undetectable, noting that kernel packets are *"not marked with the `LLMHF_INJECTED` or
`LLMHF_LOWER_IL_INJECTED` flags"* — which implies user-mode ones are.

## ⚠️ Two practical rules that belong beside any `SendInput` advice

1. **UIPI: injection fails SILENTLY across an integrity boundary.** Input may only be injected into a
   process at an equal or lesser integrity level, and when it is blocked **neither the return value
   nor `GetLastError` reports it** `[reported]`. Any harness must run at the same integrity level as
   the game — and "no effect" must not be read as "the game ignores injected input" until that is
   checked.
2. **Windows pointer ballistics scale injected mouse deltas by up to 4×**, depending on the pointer
   speed and the two threshold values `[reported, Microsoft's own documentation]`. **An injected `dx`
   is therefore not a portable unit** — a figure that worked on one machine may not on another. Pin
   the values via `SystemParametersInfo` at harness start, or calibrate against a read-back.
   This one has teeth for us: a measured "120 steps of `dx=40`" from one project was about to be
   copied to a sibling as if it were a property of the engine.

## ⚠️ And one thing that is genuinely unresolved in public sources — recorded as unresolved

**Whether `SendInput` reaches a pure Raw Input consumer is contested**, and the library should say so
rather than pick a side:

- **Against:** remote-desktop/streaming projects report their `SendInput` path producing absolute
  packets or zero deltas in raw-input games, and requiring kernel HID injection instead; one
  project's own documentation says the Windows cursor may move "even though a game that listens only
  for Raw Input receives nothing".
- **For:** the entire user-mode injection ecosystem targets raw-input shooters, and the anti-cheat
  literature treats user-mode injection as *working but detectable* — which is only coherent if the
  events arrive.

Plausible reconciliation: the streaming failures are about **injection shape** (absolute coordinates,
or relative deltas defeated by the game's own cursor clamping) rather than a hard OS rule
`[hypothesis]`. Either way, **a null result against a raw-input game is not self-explanatory** and
needs a control.

## Why this matters to the estate specifically

Two projects here have recorded DirectInput-related input conclusions that this reframes — one whose
keyboard "goes through DirectInput 8, which our key-state hooks do not feed", and one where relative
mouse movement was disproved as a camera driver against a good control. **Neither conclusion is
challenged by this** — both were about *hooking* or about a game-side behaviour, not about whether
DirectInput can see injected input — but the *reason* future sessions give for those results should be
the accurate one, not the folk memory.

## Suggested shape

A short subsection under the library's input/automation material: the wrapper fact, the
scancode-vs-virtual-key split as the reconciliation of our two contradictory first-hand results, the
three filtering APIs with their OS-version floors, and the two practical rules (UIPI silence, pointer
ballistics). The unresolved raw-input question stated as unresolved.

## Credit

**Microsoft Learn** — the DirectInput high-DPI mouse guidance (the `WM_INPUT` thread), `SendInput`,
`MOUSEINPUT` (pointer ballistics), `MSLLHOOKSTRUCT`, `GetCurrentInputMessageSource`, UIPI.
**learncodebygaming** — `pydirectinput` and the scancode requirement.
**changeofpace** — `MouClassInputInjection` (the injected-flag observation).
**ClassicOldSong** (Apollo) and the **LizardByte / Sunshine** team — the raw-input failure reports.
Our own `enslaved-vr`, `alan-wake-vr`, `doom-2016-vr` and `psychonauts-vr` control profiles for the
first-hand results being reconciled.

No page carried text addressed to AI agents. Nothing cloned, downloaded or copied.
