# Techniques

Deep dives on the parts of a flat→VR mod that recur across engines and cause the most trouble.
Each is distilled from public projects credited in
[`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).

- [Frame timing: Reflex-marker vs two-hook](#frame-timing)
- [Stereo submission: native, synchronized-sequential, AFR, AER](#stereo-submission-strategies)
- [Dormant native stereo paths (check before you build)](#dormant-native-stereo-paths)
- [Temporal effects under AFR (the TAA problem)](#temporal-effects-under-afr)
- [Basis & handedness (why the world "swims")](#basis--handedness)
- [Telling the main camera from shadow/reflection cameras](#main-camera-discrimination)
- [HUD & UI in VR](#hud--ui-in-vr)
- [Packed/self-protecting binaries](#packedself-protecting-binaries)
- [Launching a Steamworks game directly](#launching-a-steamworks-game-directly)
- [Driving a live game from a hook (harness tick sites)](#driving-a-live-game-from-a-hook)
- [The void behind the player (and how to measure it without a headset)](#the-void-behind-the-player)
- [Finding the camera matrix the engine actually reads](#finding-the-camera-matrix-the-engine-actually-reads)
- [VR body height: the HMD-anchored float](#vr-body-height-the-hmd-anchored-float)
- [Silent no-ops: verification that cannot see the failure](#silent-no-ops-verification-that-cannot-see-the-failure)
- [Hook to acquire a handle the API will not give you](#hook-to-acquire-a-handle-the-api-will-not-give-you)
- [Setting a gate before the process can guard it](#setting-a-gate-before-the-process-can-guard-it)
- [Injected input: measure it against a control](#injected-input-measure-it-against-a-control-never-against-zero)
- [Tool defaults that fabricate false negatives](#tool-defaults-that-fabricate-false-negatives)

---

## Frame timing

The single hardest milestone. The goal is always the same: drive the runtime callbacks
(`on_wait_rendering`, `on_begin_rendering`, `update_hmd_state`) at the right instants and keep
engine/render/present frame counters aligned. Engines expose **different signals** to do it:

- **Reflex-marker style** (seen in `starfield2vr` / Creation Engine 2): the engine already
  instruments its loop with NVIDIA Reflex markers, so the adapter hooks the marker function and
  decodes markers to a timeline — reusing existing instrumentation as a free, reliable clock.
- **Two-hook style** (seen in `anvilengine2vr` / Anvil): no markers to lean on, so the adapter
  hand-finds and hooks **two engine functions** — "begin engine frame" and "begin render frame"
  — and drives the callbacks + counters from those.

Lesson: your engine's timing model is dictated by what signals it exposes. Look first for
existing instrumentation (Reflex, ETW, a frame-index global); fall back to hooking the engine's
own frame-begin functions.

## Stereo submission strategies

From best-quality to most-compatible:

1. **Native stereo** — drive the engine's built-in stereo path. This is what UEVR's "Native Stereo"
   mode does on Unreal, and what REFramework does on RE Engine (whose OpenVR path ships in the
   engine). **Don't assume only modern Unreal-class engines qualify** — see
   [dormant native stereo paths](#dormant-native-stereo-paths) below.
2. **Synchronized sequential** — render both eyes on the *same* engine tick; fixes many effect
   bugs at a performance cost (a UEVR mode).
3. **AFR (alternating frame rendering)** — one eye per engine tick; per-eye framerate is halved.
   The default for closed engines that render the world once per frame.
4. **AER (alternating eye rendering)** — an AFR refinement (Luke Ross R.E.A.L.): render one eye,
   **reproject** the other from the previous frame (L, R+reproj-L, L+reproj-R, …), with
   "yaw folding"/camera-rotation compensation for smooth turning. The answer when an engine
   *refuses* to draw the world twice per frame and you can't hit 2× framerate. AER 2.0 largely
   fixed early ghosting. Dedicated page: [per-game native mods & AER](../per-game-native-mods/).

## Dormant native stereo paths

Before committing to building stereo yourself, check whether the engine already contains it. Three
public examples show this is not rare:

- **Unreal** ships a live native stereo path; UEVR activates it through reflection.
- **RE Engine** ships an OpenVR path in the engine itself; REFramework switches it on.
- **id Tech 6** (DOOM, 2016) carries a **complete but unexposed** stereo-3D subsystem — mode enum,
  separation/eye-swap/GUI-offset cvars, and an AFR-vs-both-eyes toggle — inherited from the Doom 3
  BFG generation and reachable by nothing in the shipping game's UI.

The third case is the instructive one, because it is found **statically, by string inspection alone**,
in an engine with no VR-runtime strings anywhere in it. See the
[id Tech 6 case study](../case-studies/id-tech-6-dormant-stereo.md) and the widened search heuristic
in [`engines-index.md`](../engines-index.md#how-to-identify-an-unknown-engine-static-no-launch-needed).

**Three cautions before you get excited:**

1. **Compiled-in is not the same as reachable.** Strings prove the code was built into the binary.
   They say nothing about whether you can *get at it*. On id Tech 6 the answer turned out to be no:
   the retail build boots into a "production mode" that registers only ~171 of the engine's many
   thousands of cvars, the `stereoRender_*` ones are **never registered at all**, and the master
   switch that would change that is gated by the same mechanism. Budget a cheap live probe — list
   the engine's cvars and search for the names — *before* planning around a dormant feature.
   And check the neighbours of any developer switch before flipping it: the cvar sitting next to
   "enable dev mode" there was *"FatalError rather than enter Dev Mode"*, defaulting to on.
2. **Vintage stereo was built for 3D TVs and shutter glasses, not HMDs.** Expect it to give you
   correct *stereo* — real binocular depth — without necessarily giving correct *per-eye positional*
   geometry. In the id Tech 6 case the engine's own doc-comment says the two stereo world views are
   *identical and centered between the eyes*, with separation applied downstream as a
   projection/screen-space step. That is a different thing from two properly offset eye views, and it
   points your override at the **projection** stage.
3. **No dormant path of that era supplies head tracking.** You inherit the plumbing — two views, two
   targets, eye swap, GUI depth — and still write all of the pose input yourself.

Even with all three caveats, a dormant path is worth finding: it hands you the engine authors' own
answers to "how do I get two views out of this renderer", which is normally the expensive part.

## Temporal effects under AFR

TAA, motion vectors, and NVIDIA-history-style features assume frame N+1 continues frame N's
viewpoint. AFR breaks that (consecutive frames are different eyes), producing a **stereo-but-
smeared** image. Two legitimate, opposite fixes seen in public adapters:

- **Keep the effect, fix it per eye** (Creation Engine 2): double-buffer the temporal history
  per eye (ping-pong the history resources by frame parity) so each eye keeps its own history.
  More work; preserves TAA/DLSS quality.
- **Disable the effect** (Anvil): byte-patch the TAA function out (and similar for letterbox).
  Cheaper; avoids the per-eye bug entirely; loses the AA.

Which is "right" is engine-dependent — fix per eye when quality matters and the history is
reachable; patch it off when it isn't worth it.

## Basis & handedness

The dominant cause of a broken first-eye injection ("the world swims when I turn my head") is a
wrong coordinate basis. Engines differ in up-axis (Y-up vs Z-up) and handedness. The standard
per-eye view composition round-trips through the runtime's basis:

```
transform = BASIS_ENGINE_TO_RUNTIME × rotation_offset × hmd_transform × eye × BASIS_RUNTIME_TO_ENGINE
view = view × transform          // (right-multiplied onto the engine's own view matrix)
```

Get the two basis matrices right for your engine first (log matrices in milestone 3 before you
ever write one in milestone 5). Watch for row- vs column-major (transpose) mismatches.

## Main-camera discrimination

You must apply VR only to the **main scene camera**, not to shadow-map or reflection cameras
(which also compute view/projection matrices). Public adapters use cheap discriminators — e.g.
**a far-plane threshold** (the main camera's far plane is large; shadow/reflection far planes are
small), or render-target size/format. Pick a signal that reliably isolates the world view on your
engine.

## Packed/self-protecting binaries

If a documented address, offset, or IAT hook doesn't match what a static file read finds — before
concluding "wrong build" or "wrong version" — check whether the entry point lands inside an
oversized or oddly-named section. That's the signature of a still-active protector stub: the real
code exists only after the process unpacks itself in memory at startup, so a static file read can
never see it, no matter how correct the address is.

The fix is to scan *live* process memory instead of the file on disk. Any DLL already loaded into
the target process (a same-name proxy DLL, or any other in-process hook) initializes before the
game's own unpacking runs and is not subject to the anti-attach defenses that may be blocking an
external debugger — because it's part of the process, not an outside observer of it. A short delay
after load (to let the unpacking stub finish), then a `VirtualQuery`-guarded scan of committed
readable+executable regions for the real opcode pattern, works where both a static file patch and
an external debugger attach fail. Worked example, including a case where 16 documented addresses
came back with zero static matches and 16-for-16 live matches once scanned correctly:
[RenderWare/Manhunt case study](../case-studies/packed-binary-live-memory-scan.md).

Anti-tamper sabotage of this kind is usually **per-site, not one blanket check** — each hooked call
may expect a different faked return value or a different side effect (a write to a polled global,
not just a return value). There is no single fix; each site needs its own disassembly and its own
targeted repair, usually cheapest as "force the branch the game itself takes on success," which
restores the game's own intended path rather than inventing new behavior.

## Launching a Steamworks game directly

Bypassing Steam's own launcher — to attach a debugger before the game's window exists, to launch
a specific one of several shipped executables, or to avoid a Desktop-Game-Theatre wedge some
titles hit under SteamVR — commonly breaks Steamworks-integrated games in a specific,
non-obvious way: **`SteamAPI_Init()` fails silently and the process exits immediately**, with no
window, no log file, and no crash entry, because the Steamworks SDK reads the running app's ID
either from a `steam_appid.txt` file beside the executable or from the `SteamAppId` environment
variable, and Steam itself only supplies that ID when it starts the process. This mechanism is
part of Valve's own Steamworks SDK (documented for local testing without the Steam client running
the game).

Two fixes, either sufficient alone: drop a `steam_appid.txt` containing the game's numeric App ID
next to the executable (leaves a file in the game's install directory), or set the `SteamAppId`
environment variable for just the launched process (writes nothing to disk). First-party
confirmation that the failure mode is exactly "instant silent exit, no diagnostic trace" — not a
crash, not a hang — comes from this account's DOOM (2016) project, which requires a direct launch
of its Vulkan-renderer executable (`DOOMx64vk.exe`, which Steam itself never launches):
[`doom-2016-vr/engine-research/`](https://github.com/TefMeister/doom-2016-vr/tree/main/engine-research), §10.
A related but distinct symptom — SteamVR's Desktop Game Theatre wedging on "Launching" for a
title started *through* Steam while a VR runtime is active, fixed the same way (launch the exe
directly) but for an unrelated reason — was hit independently on this account's Far Cry 2 project.

## HUD & UI in VR

A flat HUD stretched across a 100°+ field is unreadable and nauseating. The common recipe:
**expand the scissor/viewport** to the full eye render target (stop clipping), then **rescale the
UI** by user-tunable factors, and **suppress the head-tracked camera override while UI is
showing** (so menus don't move with your head). Dialog/letterbox overlays often need special
handling and are a frequent source of one-eye-only or letterboxed UI artifacts.

---

## Driving a live game from a hook

Once a VR mod can *read* the game every frame, the next step is usually to *act* — an automation
harness that drives the game unattended, so testing does not need a human at the keyboard. Acting
from a hook is a different problem from observing from one, and it fails in ways observation never
does.

### Where to dispatch from

**Guidance: prefer a simulation-phase hook over a render-path one.** Re-entering an engine's
command/console/script system while a frame is being drawn is a genuine hazard, and a hook whose
name is about *view* (a camera or view-calculation function) is naturally called *by the renderer*
— it satisfies every property you look for in a tick site ("game thread, once per frame, has the
player object") while sitting in the one call stack where acting is unsafe. Nothing in its name or
signature says "you are inside Draw". Prefer a hook named for simulation: an actor/entity `Tick`,
the world update, the input phase. If your only per-frame hook is on the render path, use it to
**queue**, and drain from a simulation-phase hook.

**⚠️ Be careful how strongly you state this, and do not cite XIII as proof of it.** Our XIII (2003)
project appeared to demonstrate exactly this rule — an automation harness draining a command queue
from a camera hook crashed with a GPF whose stack ran through `UGameEngine::Draw`, and moving
dispatch to `APlayerController::Tick` fixed it "outright, first try". `[disproved 2026-08-28]`
Re-arming that engine-level dispatch a day later crashed the game **again**, from `ULevel::Tick`,
with no render path anywhere in the stack. The call site was never the cause: that engine's
global `Exec` entry point is not callable from an injected hook *at all*. Moving the dispatch site
only appeared to fix things because two other, narrower dispatch objects absorbed every command
sent afterwards, so the failing path was never exercised again until it was deliberately re-armed.

### The better-supported finding

`[verified-live 2026-08-28, n=2 — two faults, two different call sites, same engine]`
On at least one UE2-era title, an **engine-wide `Exec`-style entry point is unsafe to call from an
injected hook regardless of phase.** Prefer **narrowly-scoped dispatch objects** — in XIII, the
player controller and its cheat manager, located by exported-vtable identity rather than a
hardcoded offset — and keep any engine-level dispatch behind a default-off flag. That is a smaller
claim than "avoid the render path", but it is the one the evidence actually supports, and it is
the one that would have prevented the second crash.

### Log before the call, and flush

`[verified-live 2026-08-27]` Independent of everything above, and unaffected by the correction.
The first version of that harness logged each command **after** it completed, so the crash left no
record of which command or which call died — the cause had to be inferred from telemetry stopping
on an exact tick and the process going idle. Logging **before** the call, flushed to disk, costs a
few lines and turns "something crashed" into "this exact call crashed". For any harness driving a
live game unattended, do it from the first version: the fault you are instrumenting for is
precisely the one that stops you collecting the evidence afterwards.

### The method lesson (the most transferable part)

**A fix that removes the symptom *and* stops the failing path from being exercised has proved
nothing about the cause.** Both effects are indistinguishable from outside. Before recording "X
caused it, because fixing X worked", ask whether the original failing path still runs. If it does
not, what you have is a hypothesis, and it should be written down as one — because the next
session will read a confident sentence and act on it. That is exactly how the XIII correction
above came to be needed: the note was believed, the tier was re-armed, and the game crashed again.

## The void behind the player

Turn your head in a flat game that has been given a VR view, and at some angle the world simply
stops: a hard-edged field of black with the HUD floating in it. This is not dark scenery and not a
lighting bug. The game culled and rendered a frustum for the direction its *own* camera was facing,
and the injected view has been rotated past the edge of what was ever drawn.

**Tell it apart from dark terrain by its boundary.** Unlit geometry has variation and irregular
edges. A frustum limit is a **razor-straight line** with rendered world on one side and uniform
black on the other. If you are unsure, sweep the view and watch the edge: terrain deforms, a
frustum edge sweeps rigidly with the view.

### Measure it without a headset

You do not need to wear anything to quantify this, and you should not try to judge it by eye. Feed
the mod a **synthetic head pose** — a scripted yaw sway of fixed amplitude, driven by an environment
variable, so a desk machine produces the exact motion a headset would — and score each captured
frame by **the percentage of its pixels that are near-black**. That number is comparable across
runs, across settings, and across sessions. Sample a full sway cycle and report peak *and* median;
peak is the worst moment the player experiences, median is what it feels like overall.

Two cautions learned the hard way, both of which would have produced a confident wrong answer:

- **Check that your head-pose input is actually running.** In our case head tracking was only
  updated from the VR runtime's pose pump and from one feature-gated preview path, neither active on
  a monitor — so head yaw would have been zero all run while the test appeared to pass and dutifully
  reported numbers. A test whose input is silently zero measures nothing.
- **Throttle your own logging.** A per-frame snapshot log wrote about sixty lines a second and
  thirteen megabytes in one short run, burying the handful of lines that mattered.

### Two levers, and what each is actually worth

Measured on Psychonauts (2005) across a 170° sway, fourteen frames per configuration
`[measured 2026-08-28]`:

| configuration | peak black | median |
| --- | --- | --- |
| baseline (no widen, camera untouched) | **91.8 %** | 52.3 % |
| frustum widened ×3.0 | 39.5 % | 28.7 % |
| widened ×3.0 **and** engine camera turned | **18.5 %** | **15.0 %** |

- **Widening the frustum** gives the view more already-rendered image to look into. It is cheap and
  it roughly halves the void — but it **plateaus and then reverses**: ×4.0 was no better than ×3.0
  at peak and measurably worse at the median, and pushing further eventually inverts the projection
  outright. A symmetric widen can never cover 180° behind the player, so this alone is a mitigation,
  never a fix.
- **Turning the engine's own camera** so the *engine* culls toward where the view is going closes
  the void completely — but only **within whatever clamp the game puts on its own camera** (a
  free-look limit, a chase-cam constraint). Past that clamp the game camera stops and the void
  returns.

They are complementary rather than competing, which the table shows: together they cut peak void by
80 %, and the residual is a patch of sky rather than a surrounding abyss.

### Do not rotate twice

If your injector already rotates the rendered image by head pose — most do, somewhere — then adding
a camera rotation on top turns the world at **double head speed**. That is a motion-sickness
generator, not a cosmetic bug, and it is very easy to ship because on a monitor it looks like
"tracking works". Split the responsibility explicitly and write down who owns what: in our case the
camera write owns **yaw** (the half that fixes culling, and the half with no clamp), and the
existing image-space path keeps pitch, roll and positional head motion, with yaw removed from the
pose before that correction is built.

Generalised from [`psychonauts-vr/modding-notes/`](https://github.com/TefMeister/psychonauts-vr/tree/main/modding-notes)
(`2026-08-28-void-SOLVED-camera-basis.md`, `2026-08-28-head-follow-camera-wired.md`) and
[`psychonauts-vr/dev-archive/`](https://github.com/TefMeister/psychonauts-vr/tree/main/dev-archive)
(`recon/2026-08-28-void-REPRODUCED-and-measured/`).

## Finding the camera matrix the engine actually reads

Most of the matrices you can find near a camera object are **derived outputs that nothing reads
back**. You can write them, read your value back at end of frame, and see no change in the picture
at all — which looks exactly like a timing failure and will send you off building elaborate
write-placement experiments for the wrong field. Before you conclude "wrong moment", establish
"right field".

**Test for effect, not for persistence.** A value that sticks proves nothing; a value that moves the
picture proves everything. Hold a deliberately extreme value and look at the frame.

**Identify an unknown matrix arithmetically instead of guessing.** If a candidate is a view matrix,
its translation row is the camera origin expressed in the matrix's own axes. Solve the system for
the origin and check whether the answer comes back as the negated camera position. That took one
pass over real numbers and settled a question three hypotheses had failed to settle — no engine
knowledge required, and it works on any engine.

Three things that must all be right before a camera write behaves:

1. **Snapshot once and write absolute values.** While the camera is stationary many engines do not
   rewrite this matrix at all, so a hook that rotates the current value *by* an offset every frame
   compounds — a fixed 15° hold became a continuous spin in under two seconds.
2. **Transform every column together.** Forward and previous-forward are a matched pair; letting
   them diverge breaks the frame in ways that look like an engine constraint and are not.
3. **Rotate the translation row too.** Leaving the translation on the old axes produces an error
   that *grows with angle* — clean at 2–5°, visibly sheared by 10°, wrecked past 15°. If your fix
   works at small angles and fails at large ones, this is the first thing to check.

**And the payoff worth knowing in advance:** on engines where this matrix is the real one,
**culling follows it**. Rotating it 90° rendered *less* black than not rotating it. If turning your
camera leaves a void, you are probably writing a derived copy.

`[verified-live 2026-08-28, n=1 engine]` — one bespoke D3D9-era engine; the arithmetic
identification step is engine-independent, the offsets obviously are not.

Same sources as the section above. Two of the three failures in that investigation were **our own
bugs masquerading as engine findings** (the compounding spin, and the diverging column pair) — both
produced broken images that read as evidence about the engine. See
[the method lesson](#the-method-lesson-the-most-transferable-part).

## VR body height: the HMD-anchored float

A distinctive third-person-body symptom, and one that is easy to spend weeks mis-attributing:
**in some poses the whole character floats above the ground**, feet hanging, knees straight.

The mechanism is a coupling most flat→VR conversions inherit. The game anchors the body so the
character's **head** sits at the headset. Any animation that lowers the head *relative to the body
root* — a weapon brace, a crouch, a lean, a flinch — therefore forces the engine to **lift the whole
body** to put the head back at the headset. The feet leave the floor, and the leg IK, which would
happily plant them, is given nothing to plant against.

**The diagnostic that identifies it:** the float survives every animation intervention you try. We
swapped motion lists, spoofed the weapon category, poisoned bank resolution so it resolved to the
unarmed set, and forced the target bank type — the animation demonstrably changed each time, in live
layer dumps, and the float did not move. **A symptom invariant across a whole family of
interventions is telling you the layer is wrong, not the parameter.** The confirming test was
physical and took seconds: the user crouched, lowering the real headset, and the feet planted and the
knees bent on their own.

**The fix is one line in the right place: lower the root.** Drop the pelvis bone by a fixed offset,
each frame, while the offending pose is active, composed after the game's own animation and before
its leg IK resolves. The game's existing IK then plants the feet and bends the knees by itself — it
is the automatic version of the user crouching. Ours settled at **0.175 m**, tuned by the user in the
headset and correct for two different player characters `[verified-live 2026-08-30, n=1 game, 2
characters]`.

Why it is safe, and what to check on your engine:

- **Skeleton-only.** In VR the hands and weapon are pinned to the controllers, not to the animation,
  so dropping the pelvis does not move the muzzle — aim and where shots land were confirmed
  unaffected.
- **Invisible in first person.** While aiming, the body is behind the view; you see arms. Lowering
  what you cannot see costs nothing.
- **It does not remove the brace**, only grounds it. That was an acceptable trade for us; decide it
  deliberately rather than discovering it.

This is *not* the familiar VR floor-calibration problem (a mismatch between the physical floor and
the game's floor, addressed by a global height offset — the ground that Skyrim VR's **VRIK** and
SteamVR's floor-fix tools cover). This one is pose-dependent, appears and disappears with an
animation state, and is fixed by moving the character's root inside the game rather than by
recalibrating the player.

Generalised from [`visceral-re2-vr/modding-notes/`](https://github.com/TefMeister/visceral-re2-vr/tree/main/modding-notes)
(`2026-08-30-aim-pose-and-foot-grounding-solved.md`).

## Silent no-ops: verification that cannot see the failure

Three independent cases inside two weeks, two of them in major public tools and one our own, all
share a shape worth naming: **a check that appears to confirm success while being structurally
incapable of detecting the failure.** None of them threw, logged an error, or left a wrong value
anywhere an inspector would look.

- **A truthiness test on a wrapper reads the wrapper, not the value.** In UEVR, a lookup returning
  an optional was tested with `!= 0`; an empty optional satisfies that comparison, so the
  "we found the slot" branch ran **exactly when the slot had not been found**, and the hook was
  installed into a garbage vtable slot. The log obligingly printed a failed search followed by a
  successful hook. Symptom: one eye keeping the wrong gamma, plus silent memory corruption. Fixed by
  **Remleo** in [UEVR PR #433](https://github.com/praydog/UEVR/pull/433) (2026-08-30), which also
  added a bounds check on the index and logging that distinguishes a real value from a fallback.
- **The same shape, one library down.** In REFramework, a Lua callback result type has no explicit
  boolean conversion, so an inherited templated conversion matched the `if` — turning "did the call
  succeed?" into "what did the script return?". The effect was that returning `false` from
  `on_pre_gui_draw_element` stopped suppressing the element, breaking every HUD-hiding script for
  nine days without an error anywhere. Fixed by **ErwinGunsmith** in
  [REFramework PR #1809](https://github.com/praydog/REFramework/pull/1809) (2026-08-28); introduced
  by [PR #1503](https://github.com/praydog/REFramework/pull/1503) (2026-08-19).
- **A read-back against the neutral value proves nothing.** Our own case: an argument-encoding
  mismatch meant every scalar float we wrote through an engine's reflection bridge landed as `0.0`.
  Writes of zero "verified" perfectly; every other write read back as a failure — so the broken path
  masqueraded as an engine that was refusing our values, and months of design were built on top of
  it. Details in the [RE Engine family page](../engines/re-engine.md#scalar-floats-passed-to-invoke-from-the-native-c-plugin-sdk-can-land-as-zero).

### What to do about it

- **Never verify with a value the broken path can also produce.** Probe with something a failure
  cannot manufacture — a distinctive non-zero number, a value outside the neutral range. If the only
  value that verifies is the identity value, you have not tested anything.
- **When an encoding, calling convention, or slot index is uncertain, probe it rather than reason
  about it.** Try each candidate in order, read back after each, lock the first that verifies, and
  log which one won. This costs a few lines once and removes a whole class of invisible failure.
- **Check the wrapper types in your own success tests.** `optional`, result and proxy types are
  exactly where "did it work?" quietly becomes "what came back?". Prefer the explicit accessor
  (`has_value()`, an explicit success check) over an implicit conversion.
- **Distrust a log that reads "search failed / installed successfully".** Two adjacent lines that
  contradict each other are the clearest possible signal, and the easiest to skim past.
- **Bound anything you use as an index.** UEVR's fix added a vtable-bounds check alongside the
  correctness fix — the cheap guard that turns silent corruption into a clean refusal.

## Hook to acquire a handle the API will not give you

Some objects have a rich, well-named API and **no way to obtain an instance**. Nothing enumerates
them, no parent exposes an accessor, and reflection over the type database shows the methods you
want on a type you cannot reach.

The way in is an **observe-only hook on one of that type's own methods** — any method the engine
calls routinely, a getter is ideal — that does nothing but retain the `this` pointer it was invoked
on, keyed by address and reference-counted so the engine does not free it underneath you. Run the
game for a moment and you have a table of every live instance. Then identify *yours* by a property
you already control: call an accessor on each candidate and match its result against the object you
created.

This is a general answer to "the API has no lookup", not an engine-specific trick — and it is
strictly safer than the alternatives, because the hook itself changes nothing. Keep it observe-only:
collect first, act in a separate pass, and gate the action on having positively identified your own
instance rather than acting on every instance you saw. Our own use of it acts only on objects the mod
itself spawned. Worked example: the RE Engine
[mirror render-layer case](../engines/re-engine.md#a-mirrors-real-control-panel-is-its-own-render-layer--and-you-get-it-by-hooking).

## Setting a gate before the process can guard it

When a game exposes a switch you must not trip interactively — a developer mode whose neighbouring
setting is documented as raising a fatal error on entry, a mode with a save-flagging side effect —
remember that **many engines accept the same setting at launch**, from the command line or a config
file, before the guard code that would object has run. Trying the launch-time route first, on
throwaway save data, is strictly cheaper than the interactive experiment and sometimes behaves
differently.

Alongside it, a claim-hygiene point that recurs whenever public precedent meets a first-party
reading: **date-match your evidence to your build.** Years of public reports describing a trick as
harmless may all predate a tripwire added later; a current-build reading of a hostile-looking flag
may equally be a narrower gate than its name suggests. Neither invalidates the other, and neither
settles it — the honest record is both observations, tagged, with the live test named as the thing
that would resolve them. Our worked instance is DOOM (2016)'s developer-mode question, in
[`doom-2016-vr/external-research/`](https://github.com/TefMeister/doom-2016-vr/tree/main/external-research)
(`topics/2026-08-27-devmode-enable-public-precedent-and-the-fatal-error-tension.md`).

## Injected input: measure it against a control, never against zero

**"The input API returned success" and "the game reacted" are different facts**, and the gap
between them has cost this account time on two unrelated engines:

- **UE2-era (XIII, 2003)** `[verified-live 2026-08-28, n=1]` — 600 pixels of injected `SendInput`
  mouse motion produced **0.0°** of yaw, in a session where injected *keyboard* input worked fine.
  The game takes the mouse through **DirectInput in exclusive mode**, which `SendInput` does not
  reach. Psychonauts hit the identical wall.
- **RE Engine (RE Village)** `[verified-live 2026-08-24]` — the game ignores `SendInput` outright,
  including after a real struct-layout bug was fixed and the calls started reporting success. It
  does respond to posted `WM_KEYDOWN`/`WM_KEYUP` messages.

In both cases the call site looked correct and the API said yes.

### The measurement

Do not ask the input API whether it worked. **Ask the renderer.**

1. Pick something cheap and repeatable that moves when the camera moves — a view-matrix candidate,
   the near-black fraction of a screen capture, a telemetry yaw reading.
2. **Run a no-input control first.** Measure, wait exactly as long as an injection run takes,
   measure again.
3. Score each input backend over the same duration **against that control**, not against zero.

The control is the entire point. Cameras drift on their own — idle sway, weapon bob, TAA jitter,
breathing animations — so without one, "something changed" reads as success and you will believe a
dead backend works. With one, a backend only counts if it beats the floor by a clear margin.

### Known input routes, by engine family

Compiled from our own live tests; incomplete on purpose, extend it as you measure.

| Engine family | `SendInput` | Posted window messages | Notes |
| --- | --- | --- | --- |
| Unreal Engine 2 era | **No** — exclusive DirectInput for the mouse | keyboard works | Mouse and keyboard take different routes; test them separately |
| Capcom RE Engine | **No** | **Yes** (`WM_KEYDOWN`/`WM_KEYUP`) | `[verified-live 2026-08-24]` |
| Double Fine bespoke (Psychonauts) | mouse: **no** | keyboard reaches gameplay, **not** menus | Title/credits screens need a real gamepad |
| id Tech 6 | Raw Input | **untested** | `[hypothesis]` — listed so the next session knows it is unmeasured |

### If you are already inside the process, stop pushing from outside

Where a proxy or injected DLL is already loaded, there is a better route than any external
injection API: **post the game a raw-input message carrying a sentinel handle, hook the raw-input
read function, and answer that one handle with data you fabricated.** The game never asks the OS
about it — it asks you. That sidesteps both walls above at once: a posted message needs no window
focus, and you are not travelling through the input stack that exclusive-mode capture owns.

Two implementation notes worth carrying:

- **Patch the import table rather than installing an inline trampoline.** It writes a data page
  instead of code, so it needs no disassembler and does not argue with **Control Flow Guard**,
  which is enabled on plenty of modern targets.
- **The known failure mode is bulk reads.** A game that drains input in bulk never calls the
  single-message read at all, so your hook installs perfectly and produces nothing. Log whether
  that import is even present, so the first live run can tell "the hook did not land" apart from
  "the hook landed on the function this game does not use."

`[built-not-proven 2026-08-31]` for the in-process route itself — designed and implemented against
id Tech 6, not yet confirmed to move a camera in a live game. Treat it as a plan with two known
walls already routed around, not as a proven result.

Generalised from a `doom-2016-vr` modding-session hand-off, and from the XIII and RE Village
sessions it cites.

## Tool defaults that fabricate false negatives

A recurring and expensive failure shape: **a default in a tool you are not thinking about silently
corrupts the input to a comparison, and the comparison then reports a confident, completely wrong
answer.** Two cases from this account, both of which cost real time:

- **Line endings versus a checked-in list.** `doom-2016-vr`'s Vulkan proxy has a good safety check:
  the build fails unless all 96 functions the game imports are present in the built DLL, comparing a
  checked-in list against the linker tool's output. On a fresh clone on a Windows machine with
  `core.autocrlf` enabled, the check reported **all 96 imports missing** — the repo copy arrived
  with CRLF endings while the tool emitted LF, so every line compared unequal, which is
  indistinguishable from every symbol genuinely being absent. The neighbouring line-*count* check
  passed happily, because counting does not care about endings, so one half of the verification said
  "fine" and the other said "catastrophe". `[verified-live 2026-08-31, n=1]`
- **Minimum string length versus short command names.** `strings` defaults to a minimum length of
  four, which silently drops every three-character token — exactly the vocabulary you are hunting
  when you grep a binary for console commands and cvars (`god`, `fov`, `map`, `set`). This produced
  one wrong published conclusion in this library before it was caught; see the
  [id Tech 6 case study](../case-studies/id-tech-6-dormant-stereo.md#a-method-trap-worth-stealing).

### What to do

- **Normalise at the comparison site, in the script — not in the file.** Re-saving a file with the
  right endings fixes today's clone and nothing else; the next checkout on the next machine converts
  it back. Strip carriage returns where the comparison happens. A `.gitattributes` pin is reasonable
  belt-and-braces, but the script must not depend on it, because the script is what runs on an
  unknown machine.
- **Be suspicious when two checks of the same thing disagree.** "Count matches, contents do not" is
  a near-certain signature of a normalisation problem rather than a real regression.
- **State the non-default explicitly** in any command you write down for later — `strings -n 3`,
  not `strings` — because the next reader will inherit your defaults, not your reasoning.
- **The second-order cost is the real one.** The natural reaction to an alarming verification
  failure is to distrust the code you just wrote. Check the plumbing of the test before you rewrite
  the thing it is testing.

Generalised from a `doom-2016-vr` modding-session hand-off, alongside the earlier `strings` trap
from the same project.

## Sources

- **XIII (2003) VR** (this account) — harness tick sites, the disproved render-path diagnosis, the log-before-the-call habit, and the exclusive-mode DirectInput wall that `SendInput` cannot cross; generalised out of [`XIII2003-vr/engine-research/`](https://github.com/TefMeister/XIII2003-vr/tree/main/engine-research) §9a/§9b
- **Psychonauts VR** (this account) — the void-behind-the-player characterisation and measurement method, the camera-matrix identification arithmetic, and the double-rotation trap; generalised out of [`psychonauts-vr/modding-notes/`](https://github.com/TefMeister/psychonauts-vr/tree/main/modding-notes) and [`psychonauts-vr/dev-archive/`](https://github.com/TefMeister/psychonauts-vr/tree/main/dev-archive)
- **Visceral — RE2 VR** (this account) — the HMD-anchored body float and the pelvis-drop grounding fix; generalised out of [`visceral-re2-vr/modding-notes/`](https://github.com/TefMeister/visceral-re2-vr/tree/main/modding-notes)
- **RE Village sniper scope** (this account) — the argument-encoding silent-zero case, the hook-to-acquire-a-handle pattern, and the posted-window-message input route; generalised out of [`re-village-scope-vr/modding-notes/`](https://github.com/TefMeister/re-village-scope-vr/tree/main/modding-notes)
- **DOOM (2016) VR** (this account) — the launch-time gate and the date-match-your-evidence point, the line-endings false negative, the `strings` minimum-length trap, and the in-process raw-input route; generalised out of [`doom-2016-vr/external-research/`](https://github.com/TefMeister/doom-2016-vr/tree/main/external-research) and [`doom-2016-vr/modding-notes/`](https://github.com/TefMeister/doom-2016-vr/tree/main/modding-notes)
- **Remleo** — [UEVR PR #433](https://github.com/praydog/UEVR/pull/433), the optional-truthiness/garbage-vtable-slot gamma fix (merged 2026-08-30)
- **ErwinGunsmith** — [REFramework PR #1809](https://github.com/praydog/REFramework/pull/1809), restoring the `false` return of `on_pre_gui_draw_element` (merged 2026-08-28)
- **prideslayer** and contributors — **VRIK Player Avatar** (Skyrim VR), cited only to distinguish the familiar VR floor-calibration/height-offset problem from the pose-dependent float described above: [nexusmods.com/skyrimspecialedition/mods/23416](https://www.nexusmods.com/skyrimspecialedition/mods/23416)
- **UEVR** render modes (native / synchronized-sequential / AFR) — [docs.uevr.io](https://docs.uevr.io/) · [github.com/praydog/UEVR](https://github.com/praydog/UEVR)
- **Luke Ross R.E.A.L.** — AER (alternating eye rendering); *technique reference only* — the GTA V repo is unlicensed (view-only, don't reuse code) and other titles are paid: [patreon.com/realvr](https://www.patreon.com/realvr) · [github.com/LukeRoss00/gta5-real-mod](https://github.com/LukeRoss00/gta5-real-mod)
- **starfield2vr** (mutars) — Reflex-marker timing, keep-and-fix-TAA per eye: [github.com/mutars/starfield2vr](https://github.com/mutars/starfield2vr)
- **anvilengine2vr** (mutars) — two-hook timing, disable-TAA, basis round-trip: [github.com/mutars/anvilengine2vr](https://github.com/mutars/anvilengine2vr)
- **vrframework** (Elliott Tate) — the framework these techniques are described against: [github.com/elliotttate/vrframework](https://github.com/elliotttate/vrframework)
- Inspection tools: [RenderDoc](https://renderdoc.org/) · [PIX](https://devblogs.microsoft.com/pix/)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
