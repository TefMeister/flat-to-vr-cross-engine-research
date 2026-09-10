# A published byte signature encodes a COMPILER, not a function — and assertion strings do not

**From:** `/gr` (estate sweep, 2026-09-10) · **For:** `/sr`, the cross-engine library
**Origin:** measured on `enslaved-vr` (UE3) by `/pd`, 2026-09-09, no launch. Curated into
`enslaved-vr/external-research/topics/2026-09-07b-public-ue3-locators-find-gnames-by-code-pattern-and-one-fork-ships-working-signatures.md` §5.

## The general claim

**A published prologue byte-pattern for a well-known engine function is not a property of that
function. It is a property of one compiler's exception scheme and one build's local-frame size** —
both of which change per build, per compiler version, and per optimisation setting, while the
function itself is unchanged.

**And its failure mode is the expensive kind: it returns nothing, which reads as *"the function is
absent"* rather than *"this signature is for a different compiler."*** A researcher who trusts the
pattern concludes the wrong thing about the binary, not about the pattern.

## The measurement behind it `[verified-numerically 2026-09-09, n=1 binary]`

A widely-used UE3 SDK (`unrealsdk`) skips the unstable `ProcessEvent` vtable index and instead scans
for the function's own prologue. Published shape: `push ebp` / `mov ebp,esp` / `push -1` /
`push <scopetable>` / `push <handler>` / `mov eax,fs:[0]` / `push eax` / `sub esp,0x50` / …

The same function in `Enslaved.exe` (UE3, 2013 PC port) is at `0x00580990` and begins:

```
55 8b ec 6a ff 68 d0 ca 91 01 64 a1 00 00 00 00 50 83 ec 54
```

Two constants differ: **one** `push imm32`, not two (this build uses the older `_except_handler3`
frame, so the handler comes from the scope table rather than a second push), and `sub esp,0x54`, not
`0x50` (a different local-frame size). **A scanner built from the published bytes matches exactly ONE
function in 23 MB of code, and it is the wrong one.**

**What survived the test and what did not:**

- ✅ The **strategy** transferred completely: the vtable index is not stable across titles, so detour
  the function's own address instead of taking a slot.
- ⚠️ The **invariant** transferred: "an SEH + /GS stack-cookie frame". That is a shape, not bytes.
- ❌ The **bytes** did not transfer at all.

## ⭐⭐ The better locator, where it is available: assertion strings

The same function was found **in one pass** by the assertion-string route: among the functions
bearing assertions from one named source file, the only VIRTUAL one — 1835 `.rdata` vtable slots
against **0** for every other candidate — asserting a named condition at a named source line
`[verified-numerically 2026-09-09]`.

**Why it is structurally better than a byte pattern, and this is the part worth generalising:**

1. **Compiler-independent** — the string literal is data the compiler cannot rewrite.
2. **Self-describing** — the assertion text names the source file, so you learn *what* you found,
   not just *where*.
3. **Self-validating** — the `__LINE__` immediate is a second, independent check that the match is
   the intended call site.

**Availability is the whole condition:** it needs a retail build that shipped with assertions
compiled in (`DO_CHECK` on, in UE3's spelling; every engine has its equivalent). That is commoner in
shipped games than people assume and is cheap to test for — grep the binary for source-file names.
**Where assertions are compiled out, prologue scanning is the fallback, not the first choice.**

## ⛔ A second, sharper warning: a derived number that "corroborates" published ones

The same session briefly derived a `ProcessEvent` vtable index of **64**, which sat neatly between
the two published UE3 values (APB: Reloaded 60, Rocket League 67) and therefore *looked like
corroboration*. **It was an artefact** `[disproved 2026-09-09]`: it came from treating runs of code
pointers in `.rdata` as vtables, and adjacent vtables in that binary abut with no separator, so runs
merge and every derived index shifts.

**The general lesson:** a derived value that falls plausibly between two published values is the
hardest kind of wrong number to catch, because the plausibility is doing the verification. Derive it
a second way or do not quote it.

## Suggested placement

The UE3 family page (as the worked example), plus whichever page carries **locating a known function
in a stripped binary** as a technique — the compiler-vs-function distinction and the assertion-string
route are engine-agnostic and belong there rather than under one engine.

## Credit

`unrealsdk` and the UE3 SDK-generator community for the published signature and for the
skip-the-index framing, which is the part that held. Measurement by this project's own modding lane.
