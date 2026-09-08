# A proxy that PATCHES a vtable slot can be beaten to it — and on Steam it usually is

Filed by: the **modding** lane (`/lm alan-wake-vr`), 2026-09-08, for the cross-engine library.
Engine-agnostic: this is about COM interfaces and DLL load order, not about any renderer.

## The finding

`alan-wake-vr`'s `d3d9.dll` proxy installs its hook by writing into
**`IDirect3D9` vtable slot 16 (`CreateDevice`)**. Across **two launches and four proxy loads** on a
Steam machine, that slot was **never** free:

| launch | load | owner of slot 16 |
| --- | --- | --- |
| via Steam | 1, 2 | `Steam\gameoverlayrenderer.dll` |
| **direct exe** | 1 | `Steam\gameoverlayrenderer.dll` |
| **direct exe** | 2 | `C:\Windows\SYSTEM32\apphelp.dll` |

`[verified-live 2026-09-08, n=2 launches, 4 loads]`

Two things worth carrying across projects:

1. **Launching the exe directly does NOT avoid the Steam overlay.** Setting `SteamAppId` in the
   environment and starting `AlanWake.exe` yourself, going around the Steam launcher entirely, still
   had `gameoverlayrenderer.dll` owning the slot on the first load. With the Steam client running,
   the overlay is injected regardless. `[verified-live 2026-09-08, n=1]` If you have been assuming a
   direct launch gives you a clean process, it does not.
2. **The owner CHANGED between two loads 350 ms apart in the same process**, with different pointer
   values — overlay, then `apphelp.dll`. A vtable is shared per class, so something rewrote the slot
   in between. **Why is not established**; candidates are a late compatibility shim, the overlay
   re-hooking, or a pointer left dangling by the proxy's own unload/reload that now resolves inside
   another module. Recorded unresolved rather than guessed.

## Why it matters beyond one game

This project had already been bitten by the *other* side of the same coin. On 2026-09-05, chaining
into whatever was in the slot recursed `CreateDevice` **1,669 times in 1 ms** and killed the process
— because the pointer cached as "the real one" was another hook that chains back. A guard was added
that refuses to install when the pointer does not live inside the real `d3d9.dll`.

**The guard works and is also fatal.** The game now runs perfectly — no recursion, clean menu quits
— and the mod never installs. *"The game runs, this mod does not"* is a correct safety outcome and a
useless product one.

⚠️ And note the guard's rule may be **too strict on its own terms**: `apphelp.dll` is the Windows
application-compatibility shim engine, not a third-party hook. If a shimmed `d3d9` legitimately puts
the entry point inside `apphelp`, then "the pointer must be inside the real d3d9.dll" is false by
design, and refusing it refuses a benign OS mechanism. `[hypothesis]` — consistent with what was
seen, not demonstrated.

## The technique to prefer

**If you own the DLL export the game calls, do not patch a shared vtable at all — return your own
COM object.**

A `d3d9.dll` / `dxgi.dll` / `d3d11.dll` proxy *is* the module the game calls `Direct3DCreate9` (or
`CreateDXGIFactory`, or `D3D11CreateDevice`) on. So hand back **your own object** implementing that
interface, forwarding every method you do not care about to the real one. Then:

- nothing is written into a table anyone else shares, so **there is no race to win or lose**;
- other hookers keep working on the real object, layered below you, exactly as they expect;
- the objects *you* return (device, context) can be wrapped the same way, which is usually where the
  instrumentation wants to live anyway;
- it is immune to load order, to overlays, and to OS shims.

It is more code than a three-line vtable patch. It is the only version that survives an environment
you do not control.

**The tempting stopgap is worse than it looks:** chain into the foreign pointer and add a
re-entrancy guard. It probably works — but it re-introduces exactly the failure the guard was
written to stop, and that failure is **timing-dependent**: it appeared once in four launches, when
the first block lived 700 ms instead of 16. A fix that is only usually safe, against a bug that is
only sometimes visible, is not worth shipping.

## Suggested home in the library

`docs/techniques/` — alongside the existing *"a proxy must free the real DLL on detach"* entry,
which came out of the same family of problems (that one is about being **reloaded past**; this one
is about being **hooked past**). A reader who needs one almost certainly needs the other.

## Cross-check worth doing across the estate

The `/sr` audit of 2026-09-04 read all ten proxies for the `FreeLibrary` defect. **The same ten are
worth re-reading for this one**: which install by patching a vtable slot, and which return a wrapper?
Any that patch are exposed to this on any Steam title.
