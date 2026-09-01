# Driving a game console with synthetic keys: the key is layout-dependent, and it may be a dead key

**From:** modding session, DOOM (2016) / id Tech 6, 2026-09-01
**Supersedes:** the DOOM-specific scancode values recorded 2026-08-31 (`VK_OEM_3 → 0x28`, console on
`VK_OEM_8`/`0xDF`). Neither reproduces. The general lesson below replaces them.
**Applies to:** any engine whose console is opened by the key left of `1` and driven by synthetic
keystrokes — id Tech, Unreal, Source, and every game console this portfolio has automated so far.

## Two traps, and both look exactly like a broken input backend

### 1. The virtual-key constant is not portable; the scancode is

DirectInput (and most engines' key handling) binds the **physical scancode**. The console is on
**scancode `0x29`** — the key left of `1` — on every layout. Which *virtual key* maps to that
scancode is layout-dependent, and there is no constant that is right everywhere.

Measured on one machine, layout `0x0425`:

| VK | `MapVirtualKeyA(vk, 0)` | |
|---|---|---|
| `VK_OEM_3` (0xC0) | `0x1A` | the "obvious" tilde VK — **wrong key**, types into the game |
| `VK_OEM_8` (0xDF) | **0 (unmapped)** | sends *nothing at all* — indistinguishable from a dead backend |
| `VK_OEM_7` (0xDE) | **`0x29`** | correct here, and only here |

**Rule: never carry a VK constant between machines, layouts or sessions.** Either send the scancode
`0x29` directly, or resolve it at runtime with `MapVirtualKeyA(scan, 1)` (scancode → VK) and check
the answer. An unmapped VK silently sends nothing, which is the worst failure mode available:
the API succeeds and the game never sees a key.

### 2. That key is often a DEAD KEY, and it eats your first character

On many non-US layouts the key left of `1` is a dead key (an accent that composes with the next
character). Opening the console leaves that accent pending, so **the first character of the command
you then type is silently transformed**:

- `getviewpos` arrived as `Çgetviewpos` → "Unknown command"
- `com_showCameraPosition` arrived as `*om_showCameraPosition`

Two commands that never ran, with no error attributable to input, on a session where the input
backend itself was under suspicion.

**Fix: after opening the console, send SPACE then BACKSPACE.** The space absorbs the composition;
the backspace deletes whatever it produced; the real command then types clean. Two keystrokes,
completely reliable.

### 3. A console toggle has state — make the helper symmetric

A "read a value from the console" helper that toggles the console open and closed only works if it
is *entered* with the console closed. Called with it already open, it closes the console and types
the command into the game as movement keys. Make such a helper **open → type → capture → close** as
one unit, so its pre- and post-state match and it is safe to call repeatedly.

## Why this is worth a library page

Every project here that automates a console hits all three. The cost is high and the symptom is
misleading: it presents as "the input backend does not work", which sends you to rewrite the input
layer instead of the four lines that open the console.
