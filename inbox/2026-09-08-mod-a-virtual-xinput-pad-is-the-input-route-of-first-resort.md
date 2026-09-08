# A virtual XInput pad is the input route of FIRST resort, not the last one

Filed by: the **modding** lane (`/lm`), 2026-09-08, for the cross-engine library.
Engine-agnostic: this is about how a game reads input, not about any renderer.

## The finding

On 2026-09-07, `the-evil-within-vr` recorded a hard blocker: **`SendInput` does not reach the game
at all.** Two launches settled it — with a controller unplugged and foreground verified before every
send, `Enter` would not even dismiss the photosensitivity splash. An earlier "verified" set of
keyboard bindings was withdrawn: the advances had been a connected DualSense all along. The queued
fix was to port a `GetDeviceState` injector into the game's own proxy DLL — real work, days of it.

**It was not needed.** `EvilWithin.exe` imports `XINPUT1_3`, and a **ViGEm virtual pad is that same
API with no code at all**. On 2026-09-08 it drove the game end to end
`[verified-live 2026-09-08, n=2 launches]`:

- the photosensitivity splash — *the exact screen `SendInput` could not pass*;
- the attract screen, the title menu, `CONTINUE` into Chapter 1;
- the pause menu and navigation within it;
- `EXIT` → confirm → **clean process exit**.

The game raised a *"Controller Connected — Xbox 360 controller"* toast and switched its on-screen
prompts to `(A) SELECT / (B) BACK`, so it bound the virtual pad as a real one.

The same day, on `doom-2016-vr` and `mad-max-vr`, virtual pads were also accepted as real
controllers. Three games, three engines, one route.

## Why it should be tried FIRST

1. **Zero code.** No proxy, no injector, no rebuild. `pip install vgamepad` plus the ViGEmBus driver,
   and the pad exists.
2. **It sidesteps the two things that usually kill synthetic input.** `SendInput` follows focus and
   is ignored by any game holding the mouse/keyboard in DirectInput exclusive mode. XInput is polled
   by the game itself; the OS reports a virtual pad through the same call, so **focus rules and
   DirectInput exclusivity stop mattering.**
3. **It is SAFER than a real controller.** On 2026-09-07 a physical DualSense's **stick drift walked
   the menu highlight from `CONTINUE` onto `NEW GAME`** — a destructive item — while only `Enter`s
   were being sent. A virtual pad's sticks sit at dead centre; that drift is structurally
   impossible.
4. **A "this game ignores synthetic input" finding is not safe to record until it has been tried.**
   That is what happened here: a genuine, carefully-measured negative about `SendInput` was
   generalised into "this game cannot be driven", and a large piece of work was queued off it.

**Check first:** does the executable import `XINPUT1_*`? If yes, this route is a ten-minute
experiment, and it should come before writing anything.

## ⚠️ The trap, and it nearly destroyed a save

**The first input after each pad connect is SWALLOWED.**

Measured directly on The Evil Within's pause menu: five `DPAD_DOWN` presses moved the highlight
**three** rows; two more presses in a freshly-created pad session moved **zero**. Inside a single pad
lifetime, after a settle wait, each press moved exactly one row — and the first after connect moved
none.

| step (one pad lifetime) | highlight |
| --- | --- |
| after ~6 s settle | RESTART CHAPTER |
| +1 `DPAD_DOWN` | RESTART CHAPTER — **swallowed** |
| +2 `DPAD_DOWN` | OPTIONS |
| + left stick | TITLE MENU |

The profile's keyboard-derived route said "Down ×5 to TITLE MENU". Following it and committing blind
would have put `A` on **RESTART CHAPTER**. Only capture-and-verify caught it.

**Working pattern:**
- do a whole navigation inside **ONE** pad lifetime — do not create a pad per keypress;
- open each session with a throwaway press that **cannot move a vertical list** (`DPAD_RIGHT`) to
  absorb the swallowed input;
- capture and verify the highlight before every commit, always.

**A second trap, from the same day on a different game:** hot-plugging pads mid-session makes Windows
draw *"Controller Connected"* toasts, and those toasts dominated a pixel-difference measurement so
badly that they read as the two strongest "hits" in a button probe (62× and 85× the control) while
the thing actually being measured had not moved at all. If you add pads during a run, either wait
out the toasts before measuring or measure something they cannot perturb.

## Tooling, if useful to the library

Three small tools were written for this and live in `flat-to-vr-RE-toolkit/tools/`:
`virtual-pad.py` (one pad, one action, exits), `hold-pads.py` (hold N pads alive — for features gated
on controller *count*), and `pad-session.py` (hold pads and run a scripted sequence with a screen
capture between steps). Names offered for reference, not as a dependency — the technique is the point
and it is four lines of `vgamepad`.

## Suggested home

`docs/techniques/` — as the input counterpart to the proxy-lifetime entries already there. A reader
arriving at "this game ignores my synthetic input" should find this before they start writing a
DirectInput hook.
