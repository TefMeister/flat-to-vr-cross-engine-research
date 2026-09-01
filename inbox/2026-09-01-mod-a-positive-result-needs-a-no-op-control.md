# A positive result needs a no-op control, the same way a negative needs a positive one

**From:** modding session, DOOM (2016) / id Tech 6, 2026-09-01
**Companion to:** `2026-08-31-mod-negative-results-need-a-positive-control.md` — this is the other half.

## The existing rule

Recorded 2026-08-31, after three withdrawn claims in one day: **before recording a NEGATIVE result
as fact, confirm the test could have produced a positive one.** Did the mechanism applying the
variable actually work? Was the game in a state where the effect was possible?

## The half that was missing

**Before recording a POSITIVE result as an attribution, confirm the mechanism alone does nothing.**

Concretely: we found an address that, when written every frame with a displaced value, moved the
camera *and* made the HUD, crosshair and weapon disappear. The obvious write-up is "holding this
address displaces the view and breaks the HUD."

That sentence contains two claims and the experiment only supports one. Writing into live engine
memory 60 times a second is itself an intervention — it could plausibly break HUD rendering on its
own, whatever value is written.

**The control costs one extra run:** hold the same address, every frame, at **the value it already
holds**. Result: nothing changes at all. HUD, crosshair and weapon stay exactly as they were.

Only now is the attribution earned: the HUD loss is caused by **displacing the view**, not by the
act of writing. And it is a real finding rather than a caveat, because it says something about the
engine — the HUD is positioned from the view, so it follows the camera out of frame.

## The general shape

For any intervention `f(x)` that produces an effect, run `f(x₀)` where `x₀` is the value the system
already had. If the effect persists, you have measured your own tool, not the system. This is
cheapest exactly when it matters most — the run is already set up, and it is one command.

## Where it bites hardest in this portfolio

Anywhere we write to live memory each frame: camera holds, cbuffer patching, bone-pose overrides,
motion-bank poisoning. In all of them "I wrote something and the picture changed" is ambiguous
between the value and the writing, and a no-op hold separates them for free.
