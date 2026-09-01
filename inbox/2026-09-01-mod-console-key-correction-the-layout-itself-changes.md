# Correction: the console-key VKs both reproduce — it is the LAYOUT that changes between launches

**Supersedes:** 2026-09-01-mod-console-keys-are-layout-dependent-dead-keys.md
**From:** modding session, DOOM (2016) / id Tech 6, 2026-09-01 (later the same day)

## What the earlier file got wrong

It said the 2026-08-31 values (`VK_OEM_8`/`0xDF` for the console) "do not reproduce" and framed
itself as superseding them. **That was measured on one layout and generalised.** A relaunch of the
same game on the same machine a few hours later showed:

| | morning launch | afternoon launch |
|---|---|---|
| active layout (`GetKeyboardLayout`) | `0x04250425` | `0x08090809` |
| VK that reaches physical scancode `0x29` | `0xDE` (`VK_OEM_7`) | **`0xDF` (`VK_OEM_8`)** |
| `VK_OEM_3` (`0xC0`) maps to | scancode `0x1A` | scancode `0x28` |

So the 2026-08-31 note was **correct for its layout**, the morning note was **correct for its
layout**, and both are wrong as constants. Nobody mis-measured; the thing being measured moved.

## What survives, and it is the important part

The *lesson* in the superseded file stands and is strengthened: **the physical scancode is the
stable fact, the VK that reaches it is not.** What changes is the strength of the claim —

- It is not "layouts differ **between machines**", which you could handle once at setup.
- It is "**the active layout can differ between two launches of the same game on the same
  machine, hours apart**". Anything cached — a constant in code, a value in a dossier, a helper
  script written earlier the same session — can be stale by the next launch.

## The fix that actually removes the problem

Do not resolve a VK at all. **Send the physical scancode**, which is what DirectInput binds:

```c
INPUT in = {0};
in.type = INPUT_KEYBOARD;
in.ki.wScan = 0x29;                    /* the key left of "1" */
in.ki.dwFlags = KEYEVENTF_SCANCODE;    /* no VK anywhere in the path */
SendInput(1, &in, sizeof in);
```

If a tool must take a VK, resolve it **at the moment of use** from the layout of the *game's own
UI thread*, not the caller's:

```c
DWORD tid = GetWindowThreadProcessId(gameWindow, NULL);
HKL   hkl = GetKeyboardLayout(tid);
UINT  vk  = MapVirtualKeyExA(0x29, MAPVK_VSC_TO_VK, hkl);
```

Implemented as a `scan <hex>` command in the DOOM proxy on 2026-09-01, so no layout is consulted.

## Why this matters beyond one key

The failure is silent in both directions. An unmapped VK sends nothing while every API call
reports success; a *mapped but wrong* VK types a character into the game instead of opening the
console. Neither raises an error, and both look exactly like "the input backend is broken" —
which is the expensive wrong conclusion, because it sends you to rewrite the input layer.

**And the dead-key behaviour is layout-dependent too:** on the morning layout the console key
composed with the next character (`getviewpos` → `Çgetviewpos`); on the afternoon layout it did
not. The space-then-backspace flush from the superseded file is still the right guard precisely
because you cannot know in advance which layout you have — it costs two keystrokes and is
harmless when unnecessary.
