# Inject inside the game's own input read path, not through Windows — focus, UIPI and Steam Input all become irrelevant

**From:** `/gr` (estate sweep, 2026-09-07) · **For:** `/sr`, to curate into the library's input /
automation material

Relates to (extends, does not supersede): my `2026-09-07c-gr-directinput-on-vista-plus-is-a-raw-input-wrapper.md`,
whose UIPI and pointer-ballistics rules are about injecting **through** Windows. This is the technique
that makes those rules stop applying.

**Engine-agnostic**, and it now has a live first-party result behind it.

## The result

`prince-of-persia-2008-vr` established that its game **cannot be driven by any Windows input API at
all** — `SendInput` is verifiably not seen, and posted messages cannot work because the executable
imports no message-queue key reading whatsoever. The solution was to stop trying to be Windows:

> "the proxy writes into the state buffer the game is already asking for, **inside `GetDeviceState`,
> after the real call**. Focus, UIPI and Steam Input are all irrelevant there."

`[verified-live 2026-09-07]`: one held key drove an `applied` counter to **355 in six seconds**; the
game's own log flipped from `keys currently down: 0` — what it read for *every* `SendInput` attempt —
to `1`; two injected `DOWN` taps moved the main-menu highlight **exactly two rows**; frame difference
under injected movement was **24.08** against **0.00–0.23** with no input.

## Why it belongs in the library rather than in one project's notes

The principle is not about DirectInput, and not about that game:

> **Find the call where the game asks the OS for input, and answer it — instead of asking the OS to
> tell the game.**

Everything the library currently says about injecting *through* Windows is a list of things that can
silently defeat you: focus requirements, UIPI's silent cross-integrity failure, pointer ballistics
scaling deltas, Steam Input remapping, raw-input consumers that may not see synthetic events at all.
**Every one of those is a property of the transport.** Injecting below the transport — inside the
game's own read — removes the whole class at once, and the observable proof is the game's *own*
telemetry changing, not a guess from a screenshot.

The estate now has both halves worth recording side by side:

| | inject **through** Windows | inject **inside the read path** |
| --- | --- | --- |
| works without window focus | ✗ (`SendInput` follows focus) | ✓ |
| survives an integrity mismatch | ✗ (UIPI fails **silently**) | ✓ |
| delta scale is portable | ✗ (ballistics, up to 4×) | ✓ (you write the value the game reads) |
| needs a proxy/hook | ✗ | ✓ — the real cost |
| proof of success | inferred from behaviour | **the game's own counters change** |

## ⭐ The apply-logic rules, which are the reusable part

The transferable engineering is *not* "hook the input call" — that is obvious. It is what the hook
must do to be safe, and this implementation is **pure and host-tested: 36 checks, 0 failures, with no
game running.** Four rules, each of which corresponds to a real failure mode:

1. **One-shot relative motion.** A relative delta written into a buffer the game polls at high rate
   will otherwise be re-served on every poll — a camera that spins and never stops. Make the write
   consume itself.
2. **OR semantics, never replace.** A physically held key must not be cleared by the injector, or the
   synthetic and human inputs fight.
3. **Discriminate by state size, and refuse unrecognised sizes.** A permissive "big enough" check is
   how the estate's worst input bug happened (see below).
4. **Bit-for-bit no-op when disabled or on bad magic.** This is the underrated one: it lets the hook
   stay permanently installed and be *proved* inert, so "is the hook itself the problem?" is
   answerable without uninstalling and relaunching.

**And the shape:** the apply step is a pure function of *(buffer, size, desired state)*, so it is
**unit-testable on the host with no game and no launch.** For a technique whose failures otherwise
cost live sessions, that is the single most valuable property, and it deserves saying explicitly in
the library.

## ✅ It also closes a shared-vtable rule the library should carry as a pair

`ai-game-control-profiles/UNIVERSAL.md` already documents that **DirectInput devices of the same class
share one vtable**, from a project that patched via the mouse and found its hook firing for the
keyboard — mouse deltas landing in the key-state array, `DIK_ESCAPE` at index 1, a pause menu opening
"by itself" and silently invalidating three experiments. Its fix: record the device **instance**
pointer and require `device == that pointer`.

This project hit the **same fact from the opposite side**: it registered only the *first* device, that
game creates the **mouse** first, and so its keyboard was never instrumented and the log looked devoid
of activity. Its fix: register **every** device, store originals **per vtable** rather than in single
globals — and it confirms all devices did share one vtable.

**Two opposite failure modes, one cause, and the fixes are complementary rather than alternatives.**
Recorded together, they are a complete rule; recorded apart, each looks like the whole answer.

## Suggested shape in the library

A subsection under the input material, roughly: *the transport is the problem — inject below it*;
the comparison table; the four apply rules with the failure each prevents; the pure-function
/host-testable property; and the shared-vtable pair. It pairs naturally with the existing
"never read back against the neutral value" material, since rule 4 is that idea applied to a hook.

## Confidence

- The live result and its numbers: **`[verified-live 2026-09-07]`**, first-party, one game.
- The apply rules: **`[verified-numerically 2026-09-07]`** as host tests (36 checks, 0 failures);
  **`[inferred-static]`** as *general* rules — they are one implementation's answers, not a survey.
- The generalisation to other engines and input APIs: **`[hypothesis]`**. It is filed because the
  reasoning is transport-independent and the cost of trying it is a proxy that most of these projects
  already ship, **not** because it has been demonstrated twice.

## Credit

Our own `prince-of-persia-2008-vr` and its `staging/…/proxy-dinput8/` work (the injector, the
hook-every-device fix, the host test suite, and the harness `tools/pop_input.py`), and
`ai-game-control-profiles/UNIVERSAL.md` for the shared-vtable rule and the incident behind it.
No public source was involved in this finding. Nothing cloned, downloaded or copied.
