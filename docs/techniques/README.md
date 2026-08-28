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
[`doom-2016-vr-engine-research`](https://github.com/TefMeister/doom-2016-vr-engine-research), §10.
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

## Sources

- **XIII (2003) VR** (this account) — harness tick sites, the disproved render-path diagnosis, and the log-before-the-call habit; generalised out of [`XIII2003-vr-engine-research`](https://github.com/TefMeister/XIII2003-vr-engine-research) §9a/§9b
- **UEVR** render modes (native / synchronized-sequential / AFR) — [docs.uevr.io](https://docs.uevr.io/) · [github.com/praydog/UEVR](https://github.com/praydog/UEVR)
- **Luke Ross R.E.A.L.** — AER (alternating eye rendering); *technique reference only* — the GTA V repo is unlicensed (view-only, don't reuse code) and other titles are paid: [patreon.com/realvr](https://www.patreon.com/realvr) · [github.com/LukeRoss00/gta5-real-mod](https://github.com/LukeRoss00/gta5-real-mod)
- **starfield2vr** (mutars) — Reflex-marker timing, keep-and-fix-TAA per eye: [github.com/mutars/starfield2vr](https://github.com/mutars/starfield2vr)
- **anvilengine2vr** (mutars) — two-hook timing, disable-TAA, basis round-trip: [github.com/mutars/anvilengine2vr](https://github.com/mutars/anvilengine2vr)
- **vrframework** (Elliott Tate) — the framework these techniques are described against: [github.com/elliotttate/vrframework](https://github.com/elliotttate/vrframework)
- Inspection tools: [RenderDoc](https://renderdoc.org/) · [PIX](https://devblogs.microsoft.com/pix/)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
