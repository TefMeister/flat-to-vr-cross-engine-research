# Capcom RE Engine

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** Capcom's RE Engine (Resident Evil 7 onwards, Devil May Cry 5, Monster Hunter
  Rise, and others).
- **Render API:** Direct3D 11 / Direct3D 12 (Vulkan on some platforms).
- **Known public VR path:** **turnkey** — the engine ships its own OpenVR path, and
  [REFramework](https://github.com/praydog/REFramework) (praydog and contributors) activates it.
  The best-supported non-Unreal case in this library. REFramework also exposes a Lua scripting
  API and a native C++ plugin API, which is where our own work in this family lives.

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| Resident Evil Village (2021) — picture-in-picture sniper scope — native C++ REFramework plugin | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/re-village-scope-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr) |
| Resident Evil 2 Remake (2019) — Visceral | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/visceral-re2-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`visceral-re2-vr`](https://github.com/TefMeister/visceral-re2-vr) |
| Resident Evil 2 Remake (2019) — arcade controls — retired; superseded by Visceral | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/arcade-controls-re2-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`arcade-controls-re2-vr`](https://github.com/TefMeister/arcade-controls-re2-vr) |

## Shared findings

*Seeded 2026-08-26; first filled 2026-08-31, from both RE Engine projects' own work plus public
sources. Each item carries a tag for how well it is actually known.*

### Spawning a visible object is prefab instantiation, not component assembly

`[verified-live 2026-08-28, n=1 game (RE Village), five recipes tried]` A GameObject built up at
runtime from components — create the object, add a mesh component, point it at a mesh, assign a
material — will read as fully configured on every reflectable flag and still never render. The
engine-sanctioned route is `via.Prefab`: make the instance, point it at a shipped `.pfb` path,
confirm the prefab resolved, then instantiate it into a folder at a position. The engine assembles
and registers the whole object itself, and it is born visible.

Public precedent for the same conclusion sits in alphazolam's **EMV Engine**, which supports
spawning prefabs across every RE Engine title it covers and warns that its own component-level
spawner does not cope with complicated GameObjects. Treat hand-assembly as the exception.

**Corollary worth stealing:** a title's shipped `.pfb` and `.rtex` paths can be enumerated offline
from the published file lists in Ekey's **REE.PAK.Tool**, without unpacking anything — the
game-relative path is the listed path minus the archive prefix and the trailing numeric suffix.

Generalised from [`re-village-scope-vr/external-research/`](https://github.com/TefMeister/re-village-scope-vr/tree/main/external-research)
(`topics/2026-08-29-runtime-mesh-spawning-via-prefab-instantiate.md`).

### A mirror's real control panel is its own render layer — and you get it by hooking

`[verified-live 2026-08-30, n=1 game (RE Village)]` `via.render.Mirror` itself exposes very little:
register and unregister a scene, a visibility flag, a render target. The properties that actually
matter live on the **`via.render.layer.Scene` instance the engine creates per mirror** — clipping
enable and clip plane, background colour, image quality, the layer's camera, its output render
target, size and region.

Two consequences worth carrying to any other title in the family:

1. **That layer instance cannot be looked up.** Neither the scene-view nor the scene type exposes
   any layer accessor, and a full sweep of the type database (291 render-layer types) confirmed the
   catalogue has no sky layer either. The way in is an **observe-only hook on one of the layer's own
   accessors that retains every `this` pointer it sees** — reference-counted, keyed by address — and
   then matches your own instance by comparing the mirror it returns against the mirror you created.
   "Hook to acquire a handle the API will not hand you" is not RE-specific; see
   [techniques](../techniques/README.md#hook-to-acquire-a-handle-the-api-will-not-give-you).
2. **Turning that layer's clipping off removes the clip-plane cut in the mirror image, live.** In
   our case one property retired a compromise that had shaped the design for a week.

Generalised from [`re-village-scope-vr/modding-notes/`](https://github.com/TefMeister/re-village-scope-vr/tree/main/modding-notes)
(`2026-08-30-abi-root-cause-clipping-kill-layer-control-panel.md`).

### Scalar floats passed to `invoke` from the native C++ plugin SDK can land as zero

`[measured 2026-08-30, n=1 project]` **The most expensive single bug either RE Engine project has
hit.** In the REFramework native plugin SDK, arguments travel through pointer-width argument slots.
A `float4` passed as a pointer-to-value arrives intact; a **scalar float written as raw bits into
the argument slot arrived as 0.0 on every call.** Because 0.0 is also a plausible value, every write
of 0 appeared to succeed and every other write read back as a failure — which sent a long
investigation chasing an engine behaviour that did not exist. The encoding that verified was
**double bits in the argument slot**: floats double-promote on the *argument* path just as they are
long known to on the *return* path (the familiar `get_FOV` case).

**The rule this produced:** never trust a property write without reading it back, and **never read
back against the neutral value** — pick a probe value a broken path cannot produce. The robust form
is an encoding probe: try each candidate encoding in turn, read back after each, and lock the first
that verifies. We have found no public write-up of the argument-path behaviour; this is our own
first-party measurement, published so nobody pays for it twice. The general form is in
[techniques → silent no-ops](../techniques/README.md#silent-no-ops-verification-that-cannot-see-the-failure).

### The live exposure value is readable, so a custom pass can match the game's grading

`[verified-live 2026-08-30, n=1 game (RE Village)]` `via.render.ToneMapping`'s exposure getter, on
the main camera's GameObject, tracks the engine's own auto-exposure in real time — measured gliding
from a bright exterior to a dark interior over roughly a second and a half. Anything you composite
yourself — a scope image, a picture-in-picture, an overlay — can scale its exposure from that number
instead of being tuned once and then being wrong in every other lighting condition.

### The game's tone curve is a published one, and its parameters are readable

`[measured 2026-08-30]` for the values, `[inferred-static 2026-09-03]` for the identification, `n=1
game (RE Village)`. `via.render.ToneMapping` on the main camera exposes a *triple-section* tonemap
whose fields — a linear section given as **begin + length**, a toe, a shoulder to a white point, a
contrast — carry the live values 0.22 / 0.40 / 1.33 / 1.0. That vocabulary and those numbers are
**Hajime Uchimura's GT tonemap** (Polyphony Digital, CEDEC 2017) at its published defaults to the
digit. The engine's own algebra has not been read, and several fields (`SDRToe`/`HDRToe`,
`WhiteRange`, `TonemapRange`) do not map one-to-one onto the published parameter list, so this is a
strong fingerprint rather than a proof; nothing built on it depends on the identification, only on the
measured shape. Two consequences for anyone compositing into this family:

- **The white-point fields move with the zone** (`MinWhitePoint` 5.6 → 8.0 and `WhiteRange` 0.9 → 0.8
  going indoors, same session), so a custom pass that copies the curve should read them live, as the
  exposure section above already does — the scope plugin now samples the component at ~2 Hz.
- **In GT the straight section spans `m .. m + (P − m)·l / a`**, 0.22 → 0.532 at these values, **not**
  `m .. m + l` `[verified-numerically 2026-09-03]` for our implementation of the published formula;
  the game's own span is inferred. The "length" is a fraction of the headroom.

The curve was built as *one file compiled twice* so it could be checked with no launch — the method is
on the techniques page as
[test a runtime-compiled shader without the game](../techniques/README.md#test-a-runtime-compiled-shader-without-the-game-one-file-two-compilers).
Source: [`re-village-scope-vr/engine-research/`](https://github.com/TefMeister/re-village-scope-vr/tree/main/engine-research)
(dossier §7) and `modding-notes/2026-09-03-tone-curve-gt-shoulder.md`.

### A Lua GUI callback's `false` return was broken for nine days in 2026

`[reported]` `on_pre_gui_draw_element` returning `false` is the documented way to stop an element
drawing, and it is how HUD-hiding scripts across this family work. A refactor merged as
[PR #1503](https://github.com/praydog/REFramework/pull/1503) (2026-08-19) silently broke it; it was
fixed by **ErwinGunsmith** in [PR #1809](https://github.com/praydog/REFramework/pull/1809), merged
2026-08-28. **A REFramework build from that window runs your suppression script without error and
draws the element anyway.** If a HUD-hiding script "does nothing", check the framework revision date
before debugging your own code. The cause generalises — see
[techniques → silent no-ops](../techniques/README.md#silent-no-ops-verification-that-cannot-see-the-failure).

### The framework's release is eighteen months behind its master, and the Lua fixes are on master

`[reported 2026-09-04]` REFramework's newest tagged release is still **v1.5.9.1 (2025-03-05)**, while
`master` is committed to daily — read from the GitHub releases API and the commit list on 2026-09-04.
That gap is not a curiosity for anyone writing Lua on this family; it decides which bugs you have.

The first days of September 2026 alone carried a run of **Lua data-model fixes** on master — array
element setting, array element **type confusion**, and string/number ambiguity, plus managed-array and
managed-string creation fixes made while adapting to a newer RE Engine title — none of which is in any
release. The `on_pre_gui_draw_element` fix in the section above (PR #1809, 2026-08-28) is in the same
category.

**⚠️ Corrected 2026-09-04 — "release or master" is a false choice, and the third state is the common
one here.** A project on this family answered the question and its build is **neither**: it runs a
**fork** build taken for a feature no upstream branch offers, and that fork **publishes no releases at
all**, so every release check ever run against it returns nothing `[verified-live 2026-09-04]`. **A
fork build's version is a commit date and nothing else.** Ask for the date, not the tag.

**Three practical consequences, and the third is the one that gets missed:**

- **On a release build**, a script that mishandles an array element or confuses a numeric string may be
  hitting known, already-fixed framework behaviour. Check the revision before rewriting your own code.
- **On a master build**, you get those fixes and also any regression of the week — the 2026-08 GUI
  callback break lived on master for nine days.
- **On a fork or pinned build, a fix newer than your build is not evidence you have the bug.** Whether
  an upstream fix repairs a long-standing defect or a **recent regression** decides whether an older
  build is affected at all, and a commit title cannot tell you which. The project above is pinned to a
  build from **March 2026**, months before the GUI regression window, so the one fix that would have
  argued for upgrading was one it never needed — and its standing rule against upgrading stands
  unopposed as a result.

**What to do instead of resolving the unresolvable: bound the exposure.** That project could not settle
whether September's array fixes applied to a March build, so it checked which of its own code touches
the surface — **none of the five scripts in its shipped release reads a managed array**; the exposure
is entirely in the **recon probes**, where a data-model bug returns a **wrong answer rather than an
error** `[inferred-static 2026-09-04]`. That is a materially different conclusion, reached without
answering the original question.

**So record the exact REFramework revision beside every Lua result you write down**, the same way you
would record a game patch version — and especially beside any result that came from iterating an
engine collection. On this family, "it worked yesterday" is a statement about two programs, not one.
The general form is on the techniques page as
[dating a dependency](../techniques/README.md#dating-a-dependency-a-fix-newer-than-your-build-is-not-evidence-that-you-are-affected).

✅ **Closed 2026-09-04: the sibling project has now recorded its revision**, and the result is worth
knowing because it is not the same program on both of this account's machines. One machine runs the
same **fork build (2026-03-11)** as the project above; the other runs an **upstream nightly from
2026-08-20**. Results from the two machines are results about two different frameworks, and the
project's dossier now says so beside each. The exported plugin API version is the same on both, which
is exactly why the difference is easy to miss.

**Upstream continued to move on the same surface, 2026-09-04 → 09-05** `[reported 2026-09-05]`, read
from the commit list and the pull request directly. `master` gained three Lua data-model commits —
array element setting, general array handling, and string-versus-number ambiguity — of which the
array one is the substantive change: the managed-array creation length was widened to a signed 64-bit
type so a negative length can no longer wrap into a huge allocation, index validation was centralised
into one checked helper (integer, in range), and out-of-range indexing now returns nothing or raises
instead of behaving undefinedly. **On a March fork build, none of that is present**, and the shape of
the exposure is the one described above: wrong values rather than errors, in recon code. Worth
knowing concretely, because a project on this family has independently measured its build's managed
array layout (element count and data at fixed offsets) and derives those offsets live at hand-over
rather than assuming them — which is the right defence against exactly this drift.

### 🚨 A game patch can move a `via.render.Texture` field, and the crash has no framework in its stack

`[reported 2026-09-05]` — read from the merged pull request and the repository API firsthand.
[REFramework PR #1822](https://github.com/praydog/REFramework/pull/1822) by **porlock2**, merged
2026-09-05, is the clearest public example this family has produced of a hazard that applies to every
project here.

A March 2026 title update re-laid-out `via.render.Texture` to match a sibling title's layout: the
description field moved to a different base, and the graphics-API resource container moved by 0x18.
The framework's **static offset accessors** kept reading the old positions. The symptom was a crash at
the publisher logo on startup under VR multipass, **on game worker threads with no framework frame in
the call stack**, with or without upscaling enabled — which is precisely why it read for months as
something other than a framework problem. The fix is four lines: a per-title branch selecting the new
values, measured rather than estimated.

**What every project on this engine should take from it:**

- **Those texture accessors are what render-target and upscaler code leans on.** If you are building a
  native render target, a mirror, or a second view, you are on this exact surface.
- **The offsets are per-title *and* per-game-version.** "RE Engine support" is really "these titles,
  at these versions".
- **Date your framework build against the game's patches, not only against upstream.** A build
  predating a title's layout change carries the pre-change path by definition, and no release check
  will tell you. Both of this account's builds predate the change above, and neither project works on
  the affected title — but the general exposure is the same for any title that re-lays-out a struct.

General form, including why the empty call stack is misleading:
[the version that moves is usually the game's](../techniques/README.md#-the-version-that-moves-is-usually-the-games--and-a-moved-struct-field-crashes-with-your-framework-nowhere-in-the-stack).
Credit: **porlock2** and **praydog**.

### Custom animation and locomotion levers, cross-title

`[reported]` The motion system is engine-level rather than per-game, so these public techniques
should port across titles in the family:

- **Registering your own motion lists at runtime** — build the motion-list resource, hold it, attach
  it to the actor's motion component as a dynamic motion bank, then drive a layer to play the clip.
  The trap is the motion state machine: leave it running and it re-drives the same layer in the same
  frame, so it must be paused and disabled for the duration. Root motion has to be applied by hand,
  moving both the transform and the character controller. Described from godlock2000-eng's
  **ResidentEvil2_CustomAnimationFramework_NonRTX**, whose docs folder is a reference-quality
  write-up of the motion and motion-list formats.
- **Changing movement speed without foot-sliding** — scale the motion layer's playback speed *and*
  the movement driver's returned speed together, so animation and translation stay in step
  (Junh2x's **RE9-Movement-Speed-Mod**).
- **Fire origin is a shipped enum, not something you have to build.** RE Engine titles fire from the
  camera in flat play; REFramework's per-game VR code flips an existing bullet-origin enum value so
  shots start at the weapon's muzzle joint instead. The transferable part is the shape, not the
  enum: **look for a dormant variant of the engine's own enum before writing a mechanism to replace
  it** — the same instinct that found id Tech 6's dormant stereo modes.

### What did *not* work, twice, on the player's aim pose

`[disproved 2026-08-30]` Both RE2 projects tried to replace the braced weapon-ready body pose by
attacking the animation system — swapping motion lists, spoofing the weapon category, poisoning bank
resolution so it falls through to the unarmed set, and forcing the animation's target bank type.
Every one of those changed the *animation* (confirmed in live motion-layer dumps) and none of them
fixed the symptom, because the symptom was never an animation problem. What it actually was, and the
one-line fix, is in
[techniques → the HMD-anchored body float](../techniques/README.md#vr-body-height-the-hmd-anchored-float).
Recorded here so a third project on this engine does not walk it a third time.

### ⭐⭐ And what DID work, 2026-09-05: the support hand is placed by overriding a getter, not by calling a setter

`[verified-live 2026-09-05, n=1 launch per weapon, 2 weapons]` The third attempt on the same problem,
and the one that landed. It is recorded here because the levers named are engine-level, not
title-level, and because the four negatives around it are worth as much as the positive.

**Every setter on the engine's IK controller was a dead end**, in four distinct ways
`[disproved 2026-09-05]`: the arm-fit target setter accepts a value every frame and moves nothing —
the game rewrites the target from its own source before the solve; the ARM solver kind cannot be
enabled by either the direct-ABI route or the reflective invoke route, with the enabled flag reading
back zero on every frame after; the arm target setter throws an internal exception on every index;
and the two-arm and hand solver objects are **null** on both weapons, at bind pose and while holding.
What remains enabled is arm-fit configured as a wrist solver — and that solver is reachable, just not
through its own interface.

**The lever is a managed getter on the weapon side.** Each weapon carries an *aid joint* — an anchor
on the **weapon** skeleton, not on the player — and the wrist solver reads its world matrix through a
two-deep getter chain, called **once per frame each** at equal counts. Post-hooking either getter and
editing the returned translation moves the player's wrist by exactly that amount; editing both is
additive; switching off returns it to zero, including under the game's own aim state. The solver
**snaps**, so any smoothing is the mod's to add in the value it returns.

Two engine-level details that generalise across titles here:

- **The weapon-to-hand joint constraint on the implement is the *attach* joint (the right hand), not
  the aid joint.** A decompile of the constraint update shows it reading the attach field and never
  the aid field, which is what settled *anchor vs. follower* — the aid joint is an anchor.
  `[inferred-static 2026-09-05]`
- **A `Nullable<via.mat4>` return arrives through a hidden return-buffer pointer**: at return, the
  hook's return value is a pointer to that buffer, with the has-value byte at offset 0 and the matrix
  at +0x10. Dereference once and check the byte before touching the payload.

**🚨 The trap that costs launches:** the getter is called at a point in the frame where the joint is
in a **pre-update pose** — measured at 0.18–0.20 m and about 48° away from its final pose on one
weapon, 18° on another — and the solver carries the **offset** you introduce across into final space
rather than the absolute value. So an absolute target expressed in final space misses by exactly that
gap, while still responding convincingly to a relative test. The working form is *blend in final
space, then map back*, with the mapping recomputed live each frame from the un-hooked value and the
engine's own world-matrix accessor for the joint. Full method, and the general rule about establishing
which space and moment a hooked value lives in:
[own the getter the solver reads](../techniques/README.md#-when-every-setter-is-a-dead-end-own-the-getter-the-solver-reads).

Also confirmed natively on this family the same day: the input-system force call raises and drops the
full aim state within a frame, and the attack kind fires a real shot (ammunition decremented, shoot
clip on its own layer) — so aim and fire are both drivable from a plugin without touching the
animation system. `[verified-live 2026-09-05, n=1]`

### Scoped optics are a fullscreen FOV zoom plus a GUI mask — there is nothing to reuse

`[verified-live 2026-08-22, RE Village, n=1]` Aiming down a sniper scope narrows the **primary
camera's** FOV from about 63° to **24.37°** and draws a mask-and-reticle GUI element
(`via.Transform` + `via.gui.GUI` + the game's scope component) over the whole frame. There is no
second camera, no offscreen scope render and no render texture — so a VR scope on this engine has to
create the magnified image from scratch, and the flat implementation contributes nothing but the
bore-sight reference. REFramework's own VR code already computes the true aim impact point with an
async physics raycast (`via.physics.System.castRayAsync`), which is the natural thing to sight
against. The equipped weapon is **its own GameObject**, not a component of the player, so the lens
quad's mount joint is found on the weapon. Engine-agnostic form:
[a flat game's "scope" is a fullscreen FOV zoom](../techniques/README.md#a-flat-games-scope-is-a-fullscreen-fov-zoom-and-vr-cannot-use-it).

### Finding anything: three parallel hierarchies, and same-frame ordering

Working blind through reflection, "where does X live" has **three structurally different answers**,
each a different call, and absence from one says nothing about the others: a **named joint** on the
skeleton (`get_Joints`), a **child GameObject** in the transform hierarchy (`get_ChildCount` /
`get_Child`), or a **component** on the object you are already holding (`get_Components`). Gameplay
objects carry 20–100 components, and the real driver is usually a generically-named manager rather
than the class with the feature's name in it. **"Zero children" is not evidence that nothing is
attached** — native systems routinely parent visual props by joint or by manager, which never appears
as a transform child.

Two field traps worth knowing before a batch of "no effect" results is believed: a type-mismatched
write **fails silently** inside a `pcall` guard, so filter by field type before concluding a field is
inert; and a per-frame IK or correction target may be **legitimately `nil`** outside the exact pose
context it serves, so re-test under the state the effect needs rather than under "player exists".

**On timing:** `re.on_pre_application_entry` / `re.on_application_entry` hook named engine frame
phases, and two callbacks that both fire "before rendering" can still fire in the wrong order
relative to each other. Sample the same value at an early and a late hook in one frame and compare —
the general form is in
[prove the value you are debugging is the one the feature reads](../techniques/README.md#prove-the-value-you-are-debugging-is-the-one-the-feature-reads).

## See also

- [engines index](../engines-index.md) — the "Capcom RE Engine" row.
- [techniques](../techniques/README.md) — the engine-agnostic forms of several findings above.
