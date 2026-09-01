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

### Search by VALUE, not by address, where the game will tell you the answer

On modern engines the address-based hunt has a structural problem before it has a tuning problem.
Games that write uniforms into **per-frame dynamic or ring buffers** reuse a given address for a
different object every frame, so "the bytes at this address changed between two snapshots" measures
**buffer recycling**, not the camera. `[inferred-static]` for the mechanism, which follows from how
ring allocators work — an attempt to demonstrate it directly on one engine was later withdrawn as a
broken experiment, so it is recorded here as the well-founded expectation it is, not as a measurement.

Structural filters are weak here for a second reason: **orthonormality barely narrows anything.** A
64 MB uniform buffer legitimately contains thousands of orthonormal transforms — a candidate list
filled to its 4096 cap even at a tolerance of 1e-5.

**The replacement is much stronger and needs no stable address at all: find a value you already
know, and search for that.** Where a game exposes ground truth — a position readout, a debug
overlay, a console command that prints the camera — read it, then scan memory for those floats.

The worked sequence, which found a camera in one session after the address approach had failed
`[verified-live 2026-08-31, n=2 independent positions]`:

1. Drive the console and print the camera position; put a live readout on screen if the engine has
   one.
2. **Screenshot the numbers.** They are the ground truth, and looking beats any derived metric.
3. Scan for those floats.
4. **Move the player, re-read, and scan again.** One position match is a coincidence; two matches at
   different positions is a finding.

Two matches also gets you the **layout** for free — which column holds translation, whether the
basis is row- or column-major, and which axis is up — because you can check the recovered matrix
arithmetically against the printed angles rather than by eye.

A pleasant consequence worth expecting: on engines that replicate the camera **per draw**, a value
search returns dozens of hits rather than one. That is an *advantage* for stereo, since each eye
needs its own view — but it also means poking any single address does nothing, and control belongs
on the write path or at the upstream source rather than on a copy.

Generalised from a `doom-2016-vr` modding-session hand-off, 2026-08-31.

## Read the shipped files before you attach anything

`[verified across five projects, 2026-08-26 → 2026-09-01]` The strongest single pattern this
account has: **the answer to "how does the camera reach the GPU" is very often sitting in files the
game already installed**, readable with no debugger, no capture, no launch, and — importantly — no
cooperation from the game's DRM.

Five projects on four unrelated engines, all answered statically:

| What shipped | What it gave up |
| --- | --- |
| **Loose shader bundle with reflection intact** (Avalanche / Mad Max) | 1363 DXBC shaders with their `RDEF` chunks; the per-object camera transform **named and located** — `WorldViewProjMatrix` at offset 0 of a 368-byte `InstanceConsts`, in 112 shaders — plus the shadow and light matrices identified as things *not* to touch |
| **The engine's own HLSL sources** (UE3 / Enslaved) | `Common.usf` reserves `c0`–`c3` as `ViewProjectionMatrix`, `c4` as world-space camera position, `c5` as `PreViewTranslation`, with a note that they must match the RHI's register enum. A capture question, settled by reading |
| **A named constant table in the executable's strings** (id Tech 6 / DOOM 2016) | The complete renderparm name table plus a reflection database carrying the developers' own doc-comments |
| **Shader bytecode on disk** (id Tech 5 / The Evil Within) | The layout gap bounded: 168 vertex shaders sorted into contiguous, no-MVP, and scattered-row groups, the last collapsing into ten distinct shapes with one covering fifteen of them |
| **The PE itself** (UE2 / XIII) | The single `__thiscall` every world, view and projection matrix passes through on its way to the D3D8 device |

### DRM protects the executable, not the data

This is the sub-lesson worth the most. **A Denuvo title that cannot be attached to can still have its
shader bundle read off disk in full.** One project in this account had stalled at first-injection for
exactly that reason; static shader reflection made its camera work startable without touching the
protected process at all.

Before recording "DRM blocks this project", ask what DRM actually covers. Anti-tamper protects the
binary's execution. It does not usually encrypt shipped shaders, script sources, config, reflection
tables, or asset metadata — and those are frequently where the camera is *described*, even when the
place it is *stored* is out of reach.

### Reflection names the per-object buffer; it usually cannot name the shared one

A limit worth knowing before you spend a session on it, and it showed up cleanly on the same project.
Reflection recovered rich member names for the **per-object** constant buffer, and **nothing** for the
shared per-frame one — because the engine fills that from C++ as a raw `float4[20]` array rather than
a struct, so the type record has no members to report.

**So expect this shape:** reflection hands you the per-object WVP by name, and the shared view
matrix — the one you actually want for a single per-eye injection point — has to be found **by
value**. That is not a failure of the technique. It narrows "somewhere in the renderer" to "one of
about twenty float4 slots in one named buffer", which is exactly the size of problem the
[value-search method](#search-by-value-not-by-address-where-the-game-will-tell-you-the-answer)
solves. The probe writes itself: watch that buffer for a slot that changes when the camera moves but
stays constant across draws within a frame.

Generalised from `mad-max-vr`, `enslaved-vr`, `doom-2016-vr`, `the-evil-within-vr` and `XIII2003-vr`
engine dossiers, all 2026-09-01.

## Counting events is not measuring content

A failure shape that has now produced two wrong published conclusions on two unrelated engines, and
it is subtle enough to deserve naming: **a frequency metric read as if it were a semantic one.**

- **UE3 / D3D9 (Enslaved)** `[corrected 2026-09-01]` — a capture recorded a vertex-shader constant
  register receiving **47 uploads per frame** and concluded the engine had no shared
  view-projection, since a shared value should be uploaded once. It was wrong: UE3's D3D9 RHI
  **re-applies the reserved view registers around bound-shader-state changes**, so those 47 uploads
  are 47 writes of *the same value*. The register really is the shared view-projection. Upload
  frequency was never evidence about sharedness, and the same capture actually **corroborated** the
  correct mapping once it was read for content rather than for counts.
- **id Tech 6 / Vulkan (DOOM 2016)** — a differential counted *"did the bytes at this address
  change"*, in memory where per-frame ring buffers hand a given address to a different object every
  frame. It was measuring buffer recycling. Detail in
  [search by value](#search-by-value-not-by-address-where-the-game-will-tell-you-the-answer).

**The rule: when a metric counts events, ask what a positive count would mean if the content never
changed — and what a low count would mean if it changed every time.** In both cases above the metric
had no answer to that question, which is exactly why it produced a confident wrong one.

Practical form: **record values, not just counts.** A capture that logs the first sixteen bytes
written alongside the write count costs almost nothing and makes both failures impossible. Where a
value is expensive to log, log a hash of it — you only need to know whether it *changed*.

## Stereo hazard: a setter that early-outs on an unchanged matrix

`[inferred-static 2026-09-01, UE2 / D3D8]` A clean, high-value hook can carry a trap that destroys
stereo *silently*, and this one is easy to walk into.

Where an engine funnels every world, view and projection matrix through a single transform setter,
that setter is the obvious per-eye injection point — both halves of true stereo in one function.
**But such setters commonly early-out when handed a matrix identical to the one already cached.**

The failure that produces: you write eye 1's view, then write eye 2's — and if your second write is
ever equal to the cached value (or your override is applied *after* the dirty check), **eye 2
silently inherits eye 1's view and stereo collapses to mono.** No error, no artefact, no log line;
the image simply looks flat, which reads as "the headset is not getting two views" and sends you to
the submission layer.

**Guards:**

- **Find the dirty check before you use the hook**, and write on the far side of it, or defeat it
  deliberately for the frames you are overriding.
- **Verify per-eye difference numerically, not by eye.** Compare the two eyes' matrices for
  inequality each frame; a flat image is far easier to diagnose when something has already asserted
  the two views were identical.
- The same caution applies to any cached-state setter in a graphics API wrapper, not just transforms.

Generalised from the `XIII2003-vr` dossier.

## Composition bugs that masquerade as handedness

`[verified numerically 2026-09-01, Dunia / Far Cry 2]` When a head-tracking composition comes out
wrong, the instinct is to reach for a handedness or axis-convention flip — the knob everyone knows is
fiddly on this kind of work. **That instinct hides a whole class of bug that flipping will
sometimes appear to fix, and never actually fixes.**

Two real bugs, both caught by a **numerical harness rather than by reading the code**, in a
head-tracking composition that looked correct:

1. **A position solve that mixed normalised basis rows with raw translation terms.** Arithmetically
   wrong; visually just an offset.
2. **A rotation composed as the *camera* rotation where the transform being modified is its
   inverse** — a world transform. This *"presents exactly like a handedness problem"*, so the
   obvious knob would have masked it while leaving the composition wrong, and the error would have
   resurfaced later as drift or as a fix that only works in one part of a level.

**The guards, in order of value:**

- **Test the maths numerically before testing it in a headset.** A harness that composes a known
  pose and checks the result against a hand-computed answer catches both bugs above in seconds, and
  neither is visible by reading.
- **Before flipping a sign, state which direction each matrix goes.** Camera-to-world or
  world-to-camera; view or inverse-view. Most "handedness" bugs in this account have turned out to be
  a transform used in the wrong direction.
- **Derive rather than assume.** The same project deliberately avoided hard-coding a runtime-to-engine
  axis table: it reads the game's camera basis from the matrix's own rows every frame, so the whole
  conversion reduces to one change of basis, and the camera's world position is *solved* from the
  matrix rather than assumed. That removes an entire category of convention bug instead of debugging
  it.

Related: [do not rotate twice](#do-not-rotate-twice), which is the same family — a correct rotation
applied in the wrong place.

Generalised from the `far-cry-2-vr` dossier and modding notes.

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
| id Tech 6 (DOOM 2016) | **Yes — both movement and look** | untested | `[verified-live 2026-08-31, movement n=2, look n=3 incl. a reversal]` · **DirectInput 8 non-exclusive**, so `SendInput` reaches it · needs the game **foregrounded** · links **XInput 1.4** directly |

**The discriminator is not the API family — it is exclusivity.** UE2-era XIII and id Tech 6 both use
DirectInput, and `SendInput` fails completely on one while driving the other. What separates them is
that XIII takes the mouse in **exclusive** mode, which `SendInput` cannot cross, while DOOM's DI8
reads the ordinary OS input stack that `SendInput` feeds. Record exclusivity, not just the API name.

**Add an "imports XInput?" note as you extend this table.** Where a game links XInput directly, a
**ViGEmBus virtual gamepad** is likely the *strongest* route rather than the fallback: the game sees
a genuine controller, movement and look sit on the sticks, and DirectInput's exclusive mode never
enters into it. That turns "which backend do I try first" from a guess into a lookup.

### Read the import table before you design the input layer

One `llvm-objdump -p` (or equivalent) tells you which input API the game actually calls, and that
decides the entire approach. It is the highest-leverage two minutes available in this area.

**The worked failure** `[disproved 2026-08-31]`: an entire in-process backend was designed and built
around posting `WM_INPUT` and answering `GetRawInputData` for DOOM (2016) — a game that imports
**zero** raw-input functions in either shipped executable. The premise came from reasoning about the
game's release year rather than from measurement. It was caught by a static check run *before* the
live session it would otherwise have wasted, and rebuilt the same day.

**The corollary deserves stating on its own: "the game is from year N, therefore it uses API X" is
not evidence.** DOOM (2016) sits on the same input path as XIII (2003).

### Saturate first, then tune down — a too-small injection reads exactly like failure

`[verified-live 2026-08-31]` About 5,400 pixels of injected mouse motion produced a few degrees of
yaw, and mouse-look was nearly written off as unreachable. About **36,000 pixels swung the view
fully round.** A marginal stimulus and a dead path are indistinguishable. When establishing whether
an input route works *at all*, push it far past anything you would use in practice; calibrate after.

### If you are already inside the process — a real option, with a wall this account has now hit

Where a proxy or injected DLL is already loaded, there is an appealing route: **hook the function
the game uses to read input and answer it with data you fabricated.** The game never asks the OS —
it asks you. In principle that sidesteps both walls above at once, since it needs no window focus
and never travels through the stack that exclusive-mode capture owns.

**In practice it works only if you hook the function the game actually consults, and that is the
part that fails.** `[disproved 2026-08-31, id Tech 6]` An in-process backend answering
`GetAsyncKeyState` / `GetKeyState` / `GetKeyboardState` installed perfectly and moved the player
**zero** metres, while `SendInput` moved 40 m under identical conditions — because DOOM's *gameplay*
keyboard goes through **DirectInput 8** (`CreateDevice(SysKeyboard)`, confirmed live). Those Win32
key-state calls really are in the import table; they serve menus and text entry, not movement.

**So an import being present does not mean the gameplay path uses it.** The import table tells you
which APIs are *available* to hook; only a live test tells you which one carries movement.
Instrument the device-creation call (here `DirectInput8Create` → `CreateDevice`) to log what the
game actually opens, and measure that before building the harder COM-level path.

An earlier version of this section presented in-process fabrication as strictly stronger than
`SendInput`. On this engine the reverse held. Both routes remain worth having; neither is the
default, and which one wins is a measurement.

Two implementation notes that do survive unchanged:

- **Patch the import table rather than installing an inline trampoline.** It writes a data page
  instead of code, so it needs no disassembler and does not argue with **Control Flow Guard**,
  which is enabled on plenty of modern targets.
- **Log whether the function you are hooking is even imported**, so the first live run can tell "the
  hook did not land" apart from "the hook landed on a function this game never calls" — which is
  precisely the case that cost the afternoon above.

Generalised from `doom-2016-vr` modding-session hand-offs (2026-08-31, including two of that
session's own corrections), and from the XIII and RE Village sessions it cites.

## Controls: a negative needs a positive one, a positive needs a no-op one

The single most productive thing this account did in one week was stop trusting results and start
running controls. Three rules came out of it, and they compose: **the first two protect the two
directions a result can point, and the third protects the instrument that produced it.**

### 1. Before recording a NEGATIVE as fact, confirm the test could have gone positive

`[verified-live 2026-08-31]` — from three wrong conclusions in a single session, all reconstructed
from logs afterwards, and all **setup** failures rather than analysis failures. The measurement was
accurate and the reasoning from it was valid each time; what was wrong was the state of the world
when the measurement was taken, which does not show up in the data.

Check three things: **the mechanism applying the variable actually works**, **only one thing changed
before the observation**, and **the system was in a state where the effect was possible.**

The three failures, because the shapes are recognisable:

- **A variable that was never applied.** A memory-differential technique was declared unable to
  discriminate camera motion because "walking scored the same as standing still". The walk had been
  issued through an input backend that an isolated test proved inert three minutes later. Both runs
  were the standing-still condition, so the comparison had no independent variable at all.
- **Three things changed before anyone looked.** A probe ran `control → backendA → backendB` and was
  screenshotted once, after all of it. Fifteen metres of movement was credited to backendA. It was
  backendB's.
- **A state that guaranteed a null.** A backend was written off because the player did not move. The
  player was **jammed against a wall** — and the *known-good* backend tested four seconds earlier
  had managed only 1.2 m for exactly that reason. The positive control was sitting in the log and
  was not read as the warning it was.

### 2. Before recording a POSITIVE as an *attribution*, confirm the mechanism alone does nothing

`[verified-live 2026-09-01]` The other half, and the newer one. "I wrote a displaced value into this
address every frame, and the camera moved and the HUD vanished" contains two claims and supports
one. **Writing into live engine memory sixty times a second is itself an intervention** — it could
plausibly break rendering on its own, whatever value it writes.

**The control costs one extra run: hold the address at the value it already holds.** In the worked
case nothing changed at all — HUD, crosshair and weapon all stayed. Only then is the attribution
earned, and the finding gets stronger rather than weaker, because the no-op run says something real
about the system rather than merely defending the claim.

The general shape: **for any intervention `f(x)` that produces an effect, run `f(x₀)` where `x₀` is
the value the system already had.** If the effect persists, you have measured your own tool.

This bites hardest anywhere we write to live memory every frame — camera holds, constant-buffer
patching, bone-pose overrides, animation-bank poisoning. In all of them "I wrote something and the
picture changed" is ambiguous between the value and the writing, and a no-op hold separates them for
free, because the run is already set up.

### 3. Validate the instrument before you trust either

A derived metric can be confidently wrong in a way no amount of care in the analysis recovers.

`[measured 2026-08-31]` A control-based probe reported **"no clear reaction"** for a backend that had
just walked the player fifteen metres. Control scored 274 changed matrices; that backend 235; another
134 — **both below the control.**

**The tell generalises: a backend that does nothing should score the *same* as the control, never
less. Scoring below your control means the instrument is measuring noise, not your variable.** That
check is free and would have caught it instantly. (The metric was counting "did the bytes at these
addresses change", but the addresses lived in per-frame ring buffers whose contents are reused for
unrelated data every frame — it was measuring buffer recycling.)

Two seconds of *looking* settled what two derived metrics got wrong: the game's own on-screen
waypoint distance. A near-miss worth recording alongside it — the replacement metric (mean pixel
difference between before/after captures) was **also** misleading, reporting 0.93 % for a working
backend, because by then the player was against a wall. **Two different derived metrics failed in one
session; opening the two images never did.**

### The practical guards, cheapest first

1. **Screenshot before the test, not only after.** Two seconds, and it captures the precondition.
   Every failure above would have been caught by it.
2. **Prove the manipulation separately, first.** Before using an injection, a cheat, a poke or a
   console command as the independent variable in some *other* experiment, verify in isolation that
   it does what its name says. A named command that silently does nothing is common in this work.
3. **One variable per observation.** If a sequence changes three things, observe after each.
4. **Prefer a control you can see** — an on-screen readout, a waypoint distance, a screenshot — over
   a derived number.
5. **Treat a suspiciously clean null as a red flag.** Two conditions matching almost exactly is often
   the same condition twice.
6. **Re-audit after fixing a tool.** When something turns out to have been broken, revisit every
   conclusion drawn while it was in use — not only the one that exposed it. That sweep is what found
   the third failure above, a full day after it had been recorded as fact.

Generalised from `doom-2016-vr` modding-session hand-offs, 2026-08-31 and 2026-09-01. The re-audit
habit came from the human partner asking whether earlier results might be wrong because the game was
not in the assumed state; the specific confound turned out to be a different one, but the instinct
found it.

## Never CPU-scan mapped GPU memory in place — it is write-combined

`[measured 2026-08-31]` Scanning about 96 MB of `HOST_VISIBLE` Vulkan memory for candidate matrices
took **3 minutes 45 seconds** and froze the game solid for the whole duration — no frames presented.
Effective read throughput was roughly **430 KB/s**, about three orders of magnitude below normal RAM.

**The cause is not the scan, it is the memory type.** `HOST_VISIBLE` upload memory is typically
**write-combined**: designed for streaming CPU *writes* toward the GPU, with CPU *reads* out of it
bypassing cache and defeating prefetch entirely. The scan compounded it with ~24 million small
strided reads (4-byte stride over 64-byte spans), close to the worst possible access pattern for WC.

**The fix is cheap, general, and was measured at ~56× on the same workload — 3 m 45 s down to about
four seconds:**

1. **One bulk sequential `memcpy` of each region into ordinary cached RAM, then scan the copy.**
   Sequential bulk reads are the one thing WC memory does acceptably.
2. **Widen the stride to the alignment you actually need.** Uniform-buffer matrices are at least
   16-byte aligned, so a 4-byte stride does four times the work for nothing.
3. **Cheap reject first.** A basis vector is unit length, so six multiplies eliminate almost every
   offset before any expensive check runs, and NaN fails the comparison for free.
4. **Order regions by flush count.** A per-frame uniform buffer flushes thousands of times; a static
   upload flushes once. When a budget runs out, spend it where the camera actually is. In the worked
   case one region showed 27,907 flushes against another's 2,983 and zero for the rest — and the
   camera was in the first.

**A second-order point worth as much as the speed:** a scan that freezes the game for minutes is not
merely slow, it **changes the experiment**. Nothing moves while it runs, so any differential that
depends on the game continuing has already been invalidated by the instrument. Make the scan
non-blocking before drawing conclusions from it.

Applies to any D3D or Vulkan project hunting matrices in mapped memory, which is most of this
estate. Generalised from a `doom-2016-vr` modding-session hand-off.

## Driving a game console with synthetic keys: scancodes, layouts, and dead keys

Every project here that automates a developer console hits the same three traps, and all three
present as **"the input backend does not work"** — which sends you to rewrite the input layer
instead of the four lines that open the console. Applies to id Tech, Unreal, Source, and every
console this account has automated.

### 1. The virtual-key constant is not portable; the scancode is

DirectInput, and most engines' key handling, binds the **physical scancode**. The console is on
**scancode `0x29`** — the key left of `1` — on every layout. Which *virtual key* reaches that
scancode is layout-dependent, and no constant is right everywhere.

**And it is worse than "layouts differ between machines"** `[measured 2026-09-01]`. Two launches of
the same game, on the same machine, hours apart:

| | morning launch | afternoon launch |
|---|---|---|
| active layout (`GetKeyboardLayout`) | `0x04250425` | `0x08090809` |
| VK reaching physical scancode `0x29` | `0xDE` (`VK_OEM_7`) | **`0xDF` (`VK_OEM_8`)** |
| scancode reached by `VK_OEM_3` (`0xC0`) | `0x1A` | `0x28` |

Nobody mis-measured. The thing being measured moved. **Anything cached — a constant in code, a value
in a dossier, a helper script written earlier in the same session — can be stale by the next
launch.**

**The fix that removes the problem rather than managing it: send the physical scancode and keep a VK
out of the path entirely.** In Win32 terms that is a keyboard `INPUT` with `wScan = 0x29` and
`KEYEVENTF_SCANCODE` set, no `wVk` at all. If a tool must accept a VK, resolve it **at the moment of
use, from the layout of the game's own UI thread** — `GetWindowThreadProcessId` → `GetKeyboardLayout`
→ `MapVirtualKeyExA(0x29, MAPVK_VSC_TO_VK, hkl)` — never from the caller's layout and never from a
constant written down earlier.

**Why this is the worst failure mode available:** an *unmapped* VK sends nothing while every API call
reports success, and a *mapped but wrong* VK types a character into the game instead of opening the
console. Neither raises an error. Both look exactly like a dead input backend.

### 2. That key is often a DEAD KEY, and it eats your first character

On many non-US layouts the key left of `1` is a dead key — an accent that composes with whatever
follows. Opening the console leaves the accent pending, so **the first character of the command you
then type is silently transformed**: `getviewpos` arriving as `Çgetviewpos`, `com_showCameraPosition`
as `*om_showCameraPosition`. Two commands that never ran, with no error attributable to input, in a
session where the input backend was already under suspicion.

**Fix: after opening the console, send SPACE then BACKSPACE.** The space absorbs the composition, the
backspace removes it, the real command types clean. Two keystrokes, and **do it unconditionally** —
the dead-key behaviour is layout-dependent too (it appeared on the morning layout above and not on
the afternoon one), so you cannot know in advance whether you need it, and it is harmless when you
do not.

### 3. A console toggle has state — make the helper symmetric

A "read a value from the console" helper that toggles open and closed only works if it is *entered*
with the console closed. Called with it already open, it closes the console and types the command
into the game as movement keys. Build such a helper as **open → type → capture → close** in one
unit, so its pre- and post-state match and it is safe to call repeatedly.

Generalised from `doom-2016-vr` modding-session hand-offs, 2026-09-01, including that session's own
same-day correction of its first write-up.

## Before you build it, check whether the game shipped it

Two habits, both cheap, both of which turned out to matter more than the work they replaced.

### Check for a community console-unlocker before declaring a production gate closed

Engines that ship in "production mode" register a fraction of their console vocabulary, and this
library already documents how to establish that cheaply and how much that knowledge is worth. What
the worked case added is the step that came after: **somebody had already published the key.**

For DOOM (2016), a live session established that retail registers 40 commands and 171 cvars, that the
stereo cvars are never registered, and that the master switch is itself unreachable — all correct. A
public mod for the same game re-adds the hidden interface on the **retail** build **without developer
mode**, taking it from **39 commands / 170 cvars to 290 / 6592**. Those numbers match the first-party
live measurement to within one each: two parties measuring the same gate independently.

**The heuristic:** a production-gated engine with an active modding scene very often has exactly one
tool whose whole purpose is unlocking the console, because that is the first thing every modder on
that engine wants. Spend **one search** before writing "the console is not a route" into a dossier —
try the engine or game name with *console unlocker*, *hidden cvars*, *dev mode*, *readd commands*,
*debug menu*.

**Two things such a tool gives you even if you never install it:**

- **A published interface dump.** The one found here ships its command and cvar lists as plain text —
  377 commands and over 11,000 cvar lines, **with the developers' own help text** — readable online,
  no download and no execution. That is a free symbol source, and it answered questions that would
  otherwise have needed live testing: a renderparm read/write command and a *set*-view-position
  command both turned out to be real named engine commands rather than strings of uncertain status.
- **An answer to "hidden, or never constructed?"** — the question this library's own id Tech 6 case
  study poses as the one that remains open. Thousands of cvars complete with help text cannot be
  hand-authored; they are an enumeration of structures the binary already contains. Where such a dump
  exists, the economical reading is **hidden and constructible**. Tag it `[reported]` until measured,
  but it is a strong prior.

**Caveats to carry:** such tools are frequently **closed-source and unlicensed**, which makes them
**prior art and feasibility proof, not something to study line-by-line** — the same category this
library already uses for commercial stereo drivers. Their patches are usually **build-specific**, so
compatibility must be verified rather than assumed. And check which DLL they proxy: a collision with
your own proxy is a problem, and even without one you have two things hooking early on purpose, so
run each alone first.

### Check whether the game shipped a photo mode before building a detached camera

Camera decoupling is the central problem of nearly every conversion here, and this library already
records games that hand it over (a `-freecamera` launch option; a community free-cam plugin). **The
one easiest to miss is a shipped Photo Mode**, because it is filed mentally as a screenshot toy.

The worked case has a retail, player-facing photo mode behind **no console, no dev mode and no cheat
gate** — an options checkbox — whose camera detaches from the player and flies on WASD **while the
game keeps running** (enemies track the camera; a key steps single frames). FOV is adjustable, the
HUD can be hidden, and its tuning knobs are ordinary cvars, including one reading like the maximum
distance the camera may travel — a value roughly **eighty times** larger than the safety clamp that
project had chosen for its own hand-built displacement.

**Why it is worth more than the screenshots:**

- **It proves the culling path follows the camera.** A shipped detached camera means the engine was
  *designed* to render correctly from where the player is not. That project had already observed an
  elevated camera rendering with no culling collapse and no black void, and recorded it as surprising
  good luck. It was not luck; it was a designed-in property, and one you can rely on.
- **It is a free instrument.** Entering photo mode and reading a candidate camera address tells you
  whether the address is the *view* or the *player body*, with no memory writes at all.
- **It shows you the engine's own answer to "what happens to first-person elements."** Photo modes
  routinely hide the HUD and weapon on purpose. If your displaced camera loses the HUD too, the
  engine may be doing what it was built to do rather than breaking.
- **It tells you whether a player body model even exists.** In the worked case the answer was no —
  the protagonist has no third-person model at all, decisive for any body-presence plan and free to
  learn.

Photo modes are usually restricted (completed campaigns, not the hardest difficulty, replay only).
Those restrictions limit their use as a *development instrument* but do not diminish what their
existence tells you about the engine.

Generalised from a `/gr doom-2016-vr` research hand-off, 2026-09-01.

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
- **Automated fetch versus a file too large to read.** `[measured 2026-09-01]` Asking an automated
  fetcher whether a name appears in a **695 KB, 11,103-line** alphabetically-sorted list returned
  **"not found"** — for a name that is genuinely in the file. Only the head of the alphabet had been
  read, and **nothing in the answer said so.** It reads exactly like a real negative and would have
  been recorded as one.
  **What caught it: a positive control in the same query.** The list of names to search also included
  one we had already verified live ourselves. That came back "not found" too, which is impossible,
  and the whole negative collapsed at once. This is the document-research sibling of the
  [control rules above](#controls-a-negative-needs-a-positive-one-a-positive-needs-a-no-op-one) —
  the same failure shape, in a different tool.

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

## Capturing the finished frame: the whole-frame route to a headset

For an old game whose renderer predates every VR runtime, there is a route to a headset that needs
no camera reverse-engineering at all: **take the finished back buffer each frame and hand it to the
compositor as a flat panel**, then drive the game's own view rotation from the HMD. It is the
cheapest first milestone in this whole library, and it is the one whose ceiling is most often
misjudged — so it is worth stating both halves plainly.

**The mechanics, on a pre-D3D11 renderer.** OpenXR and OpenVR have **no graphics binding for D3D8
or D3D9**. The bridge is therefore always two-sided: capture on the game's device (on D3D8, a
`CopyRects` of the back buffer into a system-memory image surface, then `LockRect` and decode), and
present on a **second, VR-owned D3D11 device** created purely to hold the swapchain or overlay
texture. Three things reliably go wrong at that seam:

- **A per-frame "here are raw pixels" overlay call can make the compositor tear the texture down and
  recreate it every frame** — which shows up as flicker, not as an error. Own **persistent,
  double-buffered textures on the HMD's own adapter** and hand the compositor a texture handle
  instead, uploading only when the capture's sequence number actually advances.
- **The back buffer is not necessarily the picture.** A windowed game may request a desktop-sized
  back buffer and render its viewport into one corner of it, so a faithful capture becomes a small
  image floating in a large black frame. Clamp the requested back-buffer size to the window's client
  area at device creation **and at every reset** — the reset path is the one that gets forgotten.
- **The readback is the entire cost, and the obvious optimisation may not work.** On one measured
  D3D8 title the GPU→CPU copy was ~2.7 ms average against roughly 0.6 ms for everything else in the
  path combined `[measured 2026-08-28, dev hardware, n=1 machine]`. Double-buffering the readback —
  copy into surface A while locking surface B from the previous interval — produced **no win**,
  because that driver stalls *inside the blit* rather than at the lock. Pipelining moves a stall; it
  only helps if the stall is where you think it is. Rate-cap the capture (one readback per ~11 ms is
  already 90 Hz) before optimising it, and treat a GPU-only shared-surface path as the real lever if
  the cost has to be removed rather than bounded.

**The ceiling, which is structural and not a matter of effort.** A finished frame is one image
rendered from one camera. Both eyes get the same pixels, so there is **no stereo depth**; and
because nothing about the headset ever enters the simulation, there is **no 6DoF and no
motion-controlled aim** — you can rotate the view the game renders, but you cannot make the game
render *from* the headset's position. Every project that starts here eventually needs a second
milestone that reaches the engine's own view and projection. Say so in the first milestone's own
notes, or the demo's success quietly sets the wrong expectation for what remains.

Worked example, hardware-verified in a Quest 3 via SteamVR: this account's
[XIII (2003) project](https://github.com/TefMeister/XIII2003-vr/blob/main/engine-research/ENGINE-DOSSIER.md),
§7–§8, which also carries the flicker, framing and profiling detail above.

## An old main loop may stop rendering the moment it loses focus

A single-threaded engine of the D3D8 era commonly polls `GetForegroundWindow()` once per loop
iteration and, if another process owns the foreground, **skips its tick entirely** and degenerates
into a short sleep — usually muting audio on the same branch. On a monitor this is a courtesy. Under
a headset it is fatal: the moment the user clicks anything else, presentation freezes within about a
second and the headset shows a dead panel. There is often **no ini setting for it**, because it is
not a setting — it is native code in the main loop.

The fix is to **lie to that one poll, in the narrowest possible scope**: IAT-hook
`user32!GetForegroundWindow` **in the executable's import table only**, returning the game's own
device window while a foreign window holds focus. Scoping it to the EXE matters — the window and
input drivers usually live in separate modules and should keep an honest view of focus, so mouse
capture and `WM_KILLFOCUS` behave normally. Gate it behind a config key and install it only when a
VR host is actually running.

Two consequences worth writing down rather than rediscovering:

- **Audio usually rides the same branch as the pause**, so keeping the tick alive keeps the sound
  alive too. For VR that is what you want; say so, because it reads as a bug otherwise.
- **Ticking is not receiving.** The engine now runs while unfocused, but synthetic keyboard input
  still follows the foreground window, so an automation session must hold foreground anyway. These
  are two different problems, and a single "it works unfocused now" claim silently conflates them.

While you are in that area: an injected VR host owns threads the OS will terminate abruptly at
process exit. On at least one title that left an unkillable process wedged in the display driver,
**holding a single-instance lock until reboot** — after which every launch attempt exited instantly
with code 0 and looked exactly like "the game will not start". Stopping the host threads from a hook
on the process-exit path fixed it. A silent instant exit is worth checking for a surviving instance
before it is investigated as a launch bug.

## Identify a resource by how it is used, not by its creation descriptor

When hunting for the buffer that carries the world transform, the tempting filter is the one
available earliest: hook resource creation and match on the descriptor — this size, this usage,
these bind flags. **That filter cannot work, and it fails in a way that looks like success.**

A real engine allocates many buffers with identical descriptors. One measured D3D11 title has a
1920-byte per-object world-MVP pool **and** an unrelated 1920-byte per-frame global buffer, told
apart only by which slot they are bound at and on which context type. Two separate defects in that
project — a wide filter flooding the shadow with decoys, and a later buffer-identity mix-up — trace
back to that same root cause. The property that actually identifies the buffer you want is **how it
is used**: bound at a specific slot, for a draw whose shader is known to carry the rows you care
about. That is only observable at the point of use, so register identities **at the draw**, and use
creation-time hooks only to record facts about a resource, never to decide it is the target.

Two corollaries, both learned expensively:

- **Check whether your read mechanism is even legal for that resource before debugging why it never
  fires.** A `D3D11_USAGE_DEFAULT` buffer with `CPUAccessFlags = 0` **cannot be `Map`ped at all**, by
  the API's own rules — so "no `Map` hook ever saw a write" was never an instrumentation gap or a
  timing problem, which is what two full rounds of work had assumed. The real CPU write path for such
  a buffer is `UpdateSubresource`, and shadowing that (handling partial-region writes via the
  destination box) is what worked. One reading of the real buffers' creation descriptors would have
  ruled the whole approach out on day one.
- **A mechanism that runs perfectly can still be reading the wrong thing.** The persistent-map
  capture in that project worked flawlessly, with zero failures across live sessions — on a decoy
  pool. "It executes cleanly" and "it supplies correct data" are separate claims needing separate
  evidence; see also
  [silent no-ops](#silent-no-ops-verification-that-cannot-see-the-failure).

Evidence:
[the-evil-within-vr](https://github.com/TefMeister/the-evil-within-vr/blob/main/engine-research/ENGINE-DOSSIER.md),
§7 and §11.

## Deferred-context renderers: finding the world, and patching it once per eye

On a D3D11 engine that records command lists on worker threads and replays them on the immediate
context, the per-draw camera transform is not where a naive frame trace looks for it. Three findings
that generalise to any command-list renderer:

- **To find where the world is actually drawn, disable a stage and see what disappears.** Skipping
  `ExecuteCommandList` on one title blacked out all scenery and character bodies while hair, lights
  and HUD survived — locating the bulk of the world in the deferred path in a single run, and
  incidentally identifying everything drawn directly on the immediate context. A destructive
  experiment behind an environment-variable gate answers a structural question faster than any amount
  of read-only tracing.
- **Patch at record time, not by replaying the list twice.** The per-eye work belongs at the draw the
  worker thread is recording: read the shadowed matrix rows, left-multiply by the constant per-eye
  `K_eye`, write the result into **your own** per-thread scratch buffer, rebind the slot, and let the
  original draw be recorded against it. Re-executing a finished command list per eye sounds cheaper
  and is considerably harder to make correct.
- **Do not assume one writer per buffer.** In live gameplay, 448,201 of 560,109 shadowed writes on
  that title were **cross-thread** — worker threads hand these buffers to each other across frames.
  A per-slot seqlock that *detects* the case and fail-safe-skips is the honest design; assuming a
  single writer is a claim about someone else's thread scheduler, and it holds on your machine right
  up until it does not. `[measured 2026-08-21; never observed genuinely concurrent, which is a
  weaker statement than safe]`

**One hazard specific to per-draw MVP patching:** a vertex shader that declares **no `SV_Position` in
its output signature at all** is not broken and is not a missed hook — its clip-space transform
happens downstream in a **domain shader**, and no amount of patching VS-bound constant buffers will
move that geometry. On a tessellated engine this is a real, bounded category (detailed skinned
character meshes are the usual occupants). Detect it, name it, and let those draws fail safe rather
than half-patching them: in a stereo build an unpatched draw does not render *mono*, it renders at
the **wrong eye's orientation**, which is far more disorienting than a missing object.

## The setting you want to change may be data, not code

Before patching an engine to change a startup behaviour, find out where the engine **reads that
behaviour from**. Games of the D3D8/D3D9 era routinely keep renderer and video-mode selection in a
registry key or an ini the engine parses at startup, and a value the engine chooses *itself* is worth
far more than the same value forced later from a hook — every downstream branch then runs
consistently with it.

A worked case: ten live tests went into forcing a fullscreen-only title into windowed mode through
its device-creation call. The engine turned out to read a video-mode index out of
`HKEY_CURRENT_USER` at startup and hand it straight to its own mode-set, with a branch that tests the
mode's exclusive-fullscreen flag. One DWORD does what the hook was fighting for. Two details worth
carrying:

- **The "value missing → write a default" branch is a free, non-destructive reset.** If the code
  creates the key when the read fails, deleting the key restores the game's own defaults with no file
  edits and nothing to back up.
- **Do not guess the index — ask the engine.** The mode table comes from the driver's runtime
  enumeration and varies by adapter, so a convention read from documentation is a hypothesis, not a
  fact. From an already-injected proxy you can call the engine's **own** mode-info function in a loop
  and log width/height/depth/flags for every index. One launch yields the whole table; guessing
  yields one bit per launch.

Related trap on the file side: **the ini the game writes is not always the ini you should edit** —
some titles delete their user config on exit and regenerate it at launch from a template, so edits to
the live file always vanish. Worked example on the
[Unreal 1–3 family page](../engines/unreal-1-3.md#input-is-alias-based-and-useful-aliases-often-ship-unbound).

Evidence:
[manhunt-2003-vr](https://github.com/TefMeister/manhunt-2003-vr/blob/main/engine-research/ENGINE-DOSSIER.md),
§4a.

## Make one launch answer many questions

On a game that must be launched, played into a scene, and observed, **a launch is the scarcest
resource in the project** — far scarcer than compile time or reasoning time. The habit that follows
is to stop testing one hypothesis per run.

- **Sweep the hypothesis space in a single run.** A windowed-mode device creation kept returning
  `INVALIDCALL` through three separate single-field fixes across three sessions. A seven-variant
  probe — every combination of the candidate presentation parameters, attempted in one launch against
  a throwaway hidden window before touching the game's real device — found the actual culprit
  immediately. Probing against a private window also means a wrong variant costs nothing.
- **Enumerate rather than sample.** Where the engine has a function that lists something (video
  modes, shaders, animation banks, console commands), call it in a loop from the hook and log the
  whole table, instead of testing the one entry you currently suspect.
- **Dump what an offline pass will need while you are already in there.** One project bounded a
  shader-coverage gap to ten distinct layouts from a reflection log, then could not finish the job
  because the log recorded only a base offset and no bytecode had ever been saved — a second launch
  for data the first could have carried for free.

A green light from a validation API does not substitute for any of this. `CheckDeviceType` reporting
a format/windowed pairing as valid says nothing about the *other* presentation constraints on the
same call, and it will happily bless the exact call that then fails.

## Remove your own code before accepting the blame — then fix the producer

Two rules about crashes in a game you are injecting into, both of which save days.

**A crash in a modded game is not evidence that the mod caused it.** Old titles ship with real,
reproducible defects of their own, and a stripped copy-protection layer is a particularly rich
source: protection stubs that once returned specific fake values are gone, those call sites now reach
the real Win32 APIs, and the game **punishes itself** on the failure path — stuck doors, corrupted
saves, erratic AI, hard crashes. The cheap, decisive test is not analysis, it is **absence**: rename
the proxy DLL away, reproduce the crash with your code physically not in the process, and compare the
faulting address. One such A/B produced the same fault address on the same trigger with no proxy log
written for that run — three-way agreement, and the mod was cleared in a single session instead of
being argued about for several.

**When a null pointer has many consumers, fix the producer.** Byte-patching the first crashing read on
that title worked — and the crash moved to the next consumer of the same null, then the next. Patching
consumers is unbounded, and each patch is a new invented behaviour; the producer is one site. The
related, cheaper repair when a check has been sabotaged is to **force the branch the game itself takes
when the check passes**, which restores the program's own intended path rather than inventing a new
one, and is usually a single byte.

Corollary on instrumentation: **log the pre-change value before every fix.** Two of that project's
"plausible mechanism" theories were killed in one run each by discovering that the globals they blamed
already held correct values. Without that log a wrong theory never gets falsified — it becomes
folklore in the notes.

Evidence:
[manhunt-2003-vr](https://github.com/TefMeister/manhunt-2003-vr/blob/main/engine-research/ENGINE-DOSSIER.md),
§11–§12.

## Prove the value you are debugging is the one the feature reads

The most expensive debugging sessions in this account's history were not wrong about the maths. They
were meticulously correct about a value nothing was reading.

- **Engines carry lookalike systems, and tuning the wrong one throws no error.** One weapon's VR
  reload had *three* separate position systems; the spent-shell extraction offsets were debugged to
  four decimal places and had no effect whatever on the round-insertion feature under test. Worse,
  some per-item tables have **missing entries that silently fall back to a shared default** which is
  only correct for the other items. Before deep-debugging a value, prove — by changing it grossly and
  watching for *any* effect — that the feature you are testing reads it at all.
- **An object model usually has several parallel hierarchies, and absence from one proves nothing.**
  Something attached to a character may be a named joint on the skeleton, a child in the scene
  hierarchy, or a component on the object you are already holding — three different queries, and
  "zero children" is a statement about exactly one of them. Enumerate all three before concluding
  something is not there.
- **Two callbacks that both run "before rendering" can still run in the wrong order.** Writing a value
  once per frame is only half a fix if something later in the same frame overwrites it — an IK solve,
  a derived aim vector, a re-application of the engine's own state. Do not reason about it: **sample
  the same value at an early hook and a late hook in the same frame and compare.** If they disagree,
  the ordering bug is now measured. If they agree and the effect is still wrong, it is not a timing
  problem on that value, and the next question is whether it is the right value at all.

Also worth adopting wholesale, because it costs nothing: **one bracketed tag per diagnostic script**
in every log line, so a log carrying tens of thousands of lines from every loaded script greps down to
one script's story; and **wrap every reflection call in a `pcall` equivalent**, because a native call
into an introspection surface the engine never intended will fail unpredictably, and a caught, logged
failure beats a script that dies on line one leaving a missing log line as its only symptom.

Evidence:
[visceral-re2-vr](https://github.com/TefMeister/visceral-re2-vr/blob/main/engine-research/ENGINE-DOSSIER.md),
§4, §5 and §9.

## A flat game's "scope" is a fullscreen FOV zoom, and VR cannot use it

Worth knowing before scoping any work on magnified optics — telescopic sights, binoculars, camera
viewfinders. Flat games almost never render a second view for these. The overwhelmingly common
implementation is: **narrow the main camera's FOV** (one measured case ramps ~63° down to ~24.4°,
about 2.6×) and **draw a mask-and-reticle GUI element over the whole frame**. There is no second
camera, no offscreen render target, and nothing to reuse.

In a headset both halves fail. A fullscreen FOV zoom applied to a stereo view is wrong and actively
sickening — the world's angular scale stops matching the head. And a generic VR layer that
world-positions GUI elements along the aim ray turns the fullscreen mask into a giant floating plane
in front of the player, which is the familiar "huge flat screen with a crosshair" symptom.

So a VR scope is not a port of the flat feature; it is a **new feature that must produce the magnified
image itself** — suppress the FOV ramp, hide the flat mask, render a magnified view into your own
texture, and composite that texture onto a lens quad mounted at the weapon's scope joint,
bore-sighted to whatever ray the game itself already uses for hit detection. Budget it as such.

Two probe techniques that came out of establishing the above, both reusable:

- **Recon on the flat screen first when the VR layer is HMD-gated.** With the headset off, the
  framework's VR handling stands down and the engine's *native* mechanism is visible cleanly, without
  the VR layer's compositing on top of it.
- **For a state that needs both hands on the controls, arm the probe, do not click it.** Have the
  script trigger its own dump on the state transition it is watching for (an FOV threshold crossing, a
  GUI element's draw recency) or on a timer — a diagnostic gated behind a UI button you cannot reach
  while aiming is a diagnostic that never runs.

Evidence:
[re-village-scope-vr](https://github.com/TefMeister/re-village-scope-vr/blob/main/engine-research/ENGINE-DOSSIER.md),
§2 and §5.

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
