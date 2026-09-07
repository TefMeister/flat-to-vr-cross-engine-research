# Proving BOTH eyes render, on a flat monitor, with a frame-alternating "wiggle" and a zero-noise control

*From: modding lane (`/lm alice-madness-returns-vr`), 2026-09-07, dev PC.
Engine-agnostic. Costs one launch and no extra proxy work if you already shear the projection.*

## The problem this solves

You have a stereo shear working in a D3D proxy. The picture moves when you change ipd. **That does
not prove both eyes are being produced** — a single mono view shoved sideways does exactly the same
thing, and it is the single easiest way to fool yourself into thinking a per-eye path works when
you have really only built a pan control. On a flat monitor you cannot see two eyes at once, so the
question looks like it has to wait for a headset. It does not.

## The technique

**Alternate the eye at the frame boundary in `Present`, then capture a burst of frames.**

- Flip `eye` once per `Present` — never mid-frame, so every frame is still entirely one eye and
  still the exact single-eye path your unit tests cover.
- Keep a **flip counter** in the log. This is what separates "the eye never alternated" from "the
  eye alternated and nothing moved", and those two need completely different fixes.
- Capture ~16 frames as fast as the OS will allow, then measure the horizontal displacement of each
  against the first.

**If both eyes are being produced, the frames fall into exactly TWO clusters** — never three, never a
continuum — because each capture lands on either a left-eye or a right-eye frame. On Alice: 12 px
separation at default ipd, and the two-cluster structure held at every setting tried.

### Measure displacement by cross-correlating column-mean profiles

A stereo shear is a **coherent horizontal translation**. Cross-correlating the column-mean intensity
profile measures exactly that, and is nearly blind to the things that are not translations. Mean
absolute difference is **not** a substitute — an earlier session on this same game scored input
routes by mean-luma delta and reported a *working* lever as "no effect", because animated grass and
water put the control's noise above the real signal.

## The two things that make it evidence rather than a vibe

### 1. Run the stereo-OFF control first, in the same live scene

With stereo off, 16 captures of a scene containing walking NPCs, drifting fog and idle animation
gave **spread 0 px — every single frame dx = +0**, correlation 0.984–1.000. Scene animation does not
produce a horizontal displacement. So the noise floor is not "small", it is **zero**, and any
non-zero reading afterwards is signal. A control that returns exactly zero is worth far more than
one that returns "about 3 px, probably noise".

### 2. Validate the measurement tool on synthetic offsets BEFORE trusting it

Take one real frame, shift it by known amounts, and check the tool recovers them:

```
truth  -40  -12   -3    0   +3  +12  +40
meas   -40  -12   -3    0   +3  +12  +40      7/7 exact, corr 1.0000
different-scene control:  corr 0.4556
```

Now a high correlation means "rigid translation" with evidence behind it, and you have a threshold
you can defend rather than a guess.

## Then make it quantitative: sweep ipd and fit

Two clusters prove *two eyes*. Proportionality proves it is a **baseline** and not a coincidence:

| ipd | 6.5 | 12.5 | 18.5 | 24.5 |
| --- | --- | --- | --- | --- |
| cluster separation | 12 px | 22 px | 33 px | 44 px |

```
separation = 1.7833 * ipd + 0.108 px       R^2 = 0.99948       max |residual| = 0.40 px
```

Proportional, through the origin, sub-pixel residuals. Four points and a fit is a different class of
claim from "it got bigger when I pressed the key", and it costs one extra minute.

## ⭐ The trap worth carrying to every project: a character at the convergence plane looks unsheared

Rendering the eye pair as a **red/cyan anaglyph** makes disparity visible instantly. On Alice it
immediately showed the player character with almost **no** fringing while the entire world doubled
around her — which reads exactly like the classic failure where skinned/character shaders use a
different constant register and never receive the shear. That would be a serious finding.

**It was wrong.** The discriminating test is the **convergence lever**, not the eye:

| convergence | player character | world wall | NPCs |
| --- | --- | --- | --- |
| 98 | **+78 px** | +96 | +109 |
| 300 (default) | **−1 px** | +17 | +17 |
| 915 | **+26 px** | +8 | −5 |

The character's disparity moves a long way, so she **is** being sheared — she simply sits near the
convergence distance, because a third-person camera holds the hero at a roughly fixed range. That is
correct behaviour, and on any third-person game it will look like a bug in the anaglyph.

**Generalisation:** zero disparity is ambiguous between *not sheared* and *at the convergence
plane*. Change convergence and re-measure before concluding anything. If the object moves, it is
sheared.

## Two smaller traps, both about negatives

- **A far field that is too dark to match is NOT "zero disparity".** Block-matching the distant
  street returned peaks of 0.19–0.43 — that is *no measurement*. Report it as unmeasurable and pick
  a better scene; do not record it as a depth-invariant result.
- **Check the counters actually mean what you think.** A `draws_fixed` counter here turned out to
  count *pixel*-shader fix-texture bindings, not sheared draws, so it could not corroborate anything
  about the vertex path. The frame/flip counters could, and did.

Also worth knowing: if your ipd hotkey is **F12**, that is Steam's screenshot key. Harmless to the
game, but the Steam toast overlay sits in screen captures for ~10 s and will quietly contaminate any
image analysis of that corner.

## Cost

One launch, no rebuild if the shear already exists. The whole sequence — control, four ipd settings,
a convergence sweep, and an anaglyph — took about fifteen minutes of driving, entirely from outside
the process with synthetic input and screen capture.

---

Tooling reference (BitBlt capture, scancode input with the extended-key flag, burst/shift/cluster
analysis): `alice-madness-returns-vr/dev-archive/tools/alice_harness.py`.
Full write-up: `alice-madness-returns-vr/modding-notes/2026-09-07-both-eyes-are-real-the-rock-scales-with-ipd-at-r2-0-9995.md`.
