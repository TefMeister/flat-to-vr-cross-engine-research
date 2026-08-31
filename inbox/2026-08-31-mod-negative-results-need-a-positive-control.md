# A negative result is only evidence if the test could have produced a positive one

**From:** modding session (`doom-2016-vr`), 2026-08-31
**Kind:** method, engine-agnostic
**Confidence:** `[verified-live 2026-08-31]` — three separate wrong conclusions in one session,
all reconstructed from logs.

Companion to the "verify the knob turns" drop filed the same day. That one is about the *mechanism*;
this is about the *conditions*. Together they cover every wrong conclusion this session produced.

## The rule

> Before recording a negative result as a fact, confirm the test **could** have produced a positive
> one. Check three things: the mechanism applying the variable actually works, only one thing
> changed before the observation, and the game was in a state where the expected effect was possible.

## The three failures it comes from

**1. A test whose variable was never applied.** A memory-differential technique was declared unable
to discriminate camera motion, on "walking scores the same as standing still". The walk was issued
through an input backend proven inert three minutes later. Both runs were the standing-still
condition.

**2. A test that changed three things before looking.** A probe ran `control → backendA → backendB`
and was screenshotted once, after all of it. The movement was credited to backendA; it was
backendB's.

**3. A test run in a state that guaranteed a null.** A backend was declared non-working because the
player did not move. The player was **jammed against a wall** — the *known-good* backend tested four
seconds earlier had also managed only 1.2 m, for exactly that reason. A working mechanism and a dead
one are indistinguishable when movement is impossible.

## Why this is worth its own page

In all three cases the measurement was accurate and the reasoning from it was valid. What was wrong
was **the state of the world when the measurement was taken** — which no amount of care in the
analysis can recover, and which does not show up in the data itself. A null looks the same whether
it means "no effect" or "the experiment could not have shown an effect".

## Practical guards, cheapest first

1. **Screenshot before the test, not only after.** It costs two seconds and it captures the
   precondition. Every one of the three failures would have been caught by it.
2. **Run a positive control in the same conditions.** If a known-good mechanism also produces a null
   right now, the conditions are wrong, not the mechanism under test. In failure 3 the positive
   control was *right there in the log* — 1.2 m from a backend known to move 40 m — and I did not
   read it as the warning it was.
3. **Treat a suspiciously clean null as a red flag.** Two conditions matching almost exactly is
   often the same condition twice.
4. **Re-audit after fixing a tool.** When something turns out to have been broken, revisit every
   conclusion drawn while it was in use — not only the one that exposed it. That sweep is what
   found failure 3, a full day after it was recorded as fact.
