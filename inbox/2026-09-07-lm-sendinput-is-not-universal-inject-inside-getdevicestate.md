# `SendInput` is not a universal input route — inject inside `GetDeviceState` instead

**From:** `/lm` (dev PC, 2026-09-07) · **For:** the cross-engine library — this is engine-agnostic
and cost two games a session each to learn.

## The finding

**Two unrelated games refused injected Windows input entirely on the same day**, and for the same
reason: they read the keyboard through **DirectInput8**, which does not see `SendInput`.

| game | engine | evidence |
| --- | --- | --- |
| Prince of Persia (2008) | Scimitar (D3D9, PE32) | game foreground, keyboard acquired **NONEXCLUSIVE**, polled ~200 Hz, a key **held 22 s across four logged samples**, the game's own state buffer read **0 keys down** throughout `[verified-live 2026-09-07]` |
| The Evil Within | id Tech 5 / STEM (D3D11, PE64) | with a pad unplugged and foreground verified before every send, `Enter` would not dismiss the photosensitivity splash `[verified-live 2026-09-07, n=2 launches]` |

PoP additionally imports **no message-queue key reading at all** — no `GetAsyncKeyState`,
`GetKeyboardState`, `GetKeyState`, raw input or `GetMessage`, only `PeekMessage` as the pump — so
posted `WM_KEYDOWN`/`WM_CHAR` could never have worked either. `[verified-numerically 2026-09-07]`

## The technique that does work, and it is engine-agnostic

**Hook `IDirectInputDevice8::GetDeviceState` in a proxy and OR your state into the buffer the game
is already asking for, after the real call.** Focus, UIPI and Steam Input are all irrelevant,
because it happens *inside the game's own read path*.

Proven live on PoP the same day: menus navigated, a save loaded end to end, the character driven,
the camera driven. Vtable slots on `IDirectInputDevice8`: `Acquire` 7, `GetDeviceState` 9,
`GetDeviceData` 10, `SetCooperativeLevel` 13.

Three rules the implementation earned:

1. **OR, never assign** — a key the human is physically holding must never be cleared by injection.
2. **Relative mouse motion must be consumed after one application**, or a single written delta is
   re-added at the poll rate and the camera spins forever.
3. **A counter that always rises is not evidence.** The first version incremented "applied" on
   every call once enabled; it reached 45,773 while proving nothing. Count only genuine injections.

## Three traps worth carrying to any game

- **⚠️ A connected gamepad FAKES keyboard success.** A DualSense made The Evil Within's splash,
  title and menu all advance while only `Enter` was injected — and its **stick drift moved the menu
  highlight onto `NEW GAME`**, a destructive item, unprompted. Unplug the pad before judging any
  keyboard route.
- **⚠️ Verify foreground, do not merely set it.** `SetForegroundWindow` fails silently when another
  process owns the foreground; the keys then go elsewhere and it reads as "the game ignores input".
- **⚠️ Test input where the game actually READS it.** PoP's title screen polls nothing at all, so
  every early `SendInput` test there was a negative with no power to produce a positive. It also
  runs an attract reel on an idle timer, which advanced the screen by itself and briefly looked
  like an input success.

## Reference implementation

`staging/prince-of-persia-2008-vr/proxy-dinput8/` — `input_inject.{c,h}` (36 host checks, no game
needed), a shared-memory block, and a `pop_input.py` harness. The offsets the harness writes at are
checked against the compiler's own `offsetof` output by `tools/check_offsets.py`, because a
mismatch there would not crash — it would silently press the wrong keys.

Written from scratch; no third-party hooking library.
