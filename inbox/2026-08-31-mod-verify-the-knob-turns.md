# Verify the knob turns before you trust the dial

Supersedes: extends my own 2026-08-31 drop on the attribution trap — same root cause, and it has
now produced two wrong conclusions in one session, so it deserves stating as one rule rather than
two anecdotes.

**From:** modding session (`doom-2016-vr`), 2026-08-31
**Confidence:** `[verified-live 2026-08-31]` — both failures reconstructed from the session logs.

## The rule

> **A test whose independent variable is applied through an unverified mechanism proves nothing.**
> Before believing a measurement, confirm that the condition you *think* you varied actually varied.

## Failure 1: the differential that had no movement in it

Testing whether a memory-scanning technique could discriminate camera motion:

| Run | result |
|---|---|
| snapshot → **walk** → compare | 319 survivors |
| snapshot → **stand still** → compare | 331 survivors |

Conclusion drawn: "standing still scores the same as walking, so the technique measures noise."
Written into the engine dossier as a disproved-fact.

**The walk never happened.** The movement was issued through an input backend that an isolated test
proved — *three minutes later in the same session* — moves the player **zero** units. Both runs were
the stand-still condition. Two identical conditions scoring the same is not evidence about the
technique; it is evidence the experiment had no independent variable.

## Failure 2: attributing a result to the wrong cause

A probe ran `control → backendA → backendB` in sequence, and the screenshot was taken **after the
whole sequence**. The 15 m of player movement was credited to backendA. It was backendB's. Isolated
per-backend tests, screenshotted immediately after each, reversed the conclusion in two minutes.

## Why these are one rule

Both are failures to establish that the thing being manipulated was actually being manipulated —
once by using an inert mechanism, once by changing several things before observing. In both cases
the *measurement* was fine; the **setup** was not, and no amount of care in the analysis could
recover it.

## Practical form

1. **Prove the manipulation first, separately.** Before using input injection (or a cheat, a poke,
   a console command) as the independent variable in some *other* experiment, verify in isolation
   that it does what its name says. A named command that silently does nothing is common in this
   work — see the DirectInput / SendInput / PostMessage split across engines.
2. **One variable per observation.** If a sequence changes three things, observe after each.
3. **Prefer a control you can see.** An on-screen readout (position, waypoint distance, health) or a
   screenshot beats an assumption that a command landed. This project has three separate cases where
   a derived number pointed the wrong way and a screenshot was unambiguous.
4. **Suspect a null result that is *too* clean.** Two conditions matching almost exactly is often
   not "no effect" — it is the same condition twice.

## Credit

Caught because the human partner asked whether early results might be wrong because the game was
not in the state I assumed. The specific confound turned out to be different from the one suspected,
but the instinct — *re-check the conclusions drawn while conditions were unverified* — is what found
it. Worth generalising: **after discovering that a tool was broken, revisit every conclusion drawn
while it was in use**, not just the one that exposed it.
