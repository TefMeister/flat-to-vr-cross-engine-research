# Techniques

Deep dives on the parts of a flat→VR mod that recur across engines and cause the most trouble.
Each is distilled from public projects credited in
[`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).

- [Frame timing: Reflex-marker vs two-hook](#frame-timing)
- [Stereo submission: native, synchronized-sequential, AFR, AER](#stereo-submission-strategies)
- [OpenXR carries a pose per view where OpenVR collapses to one](#openxr-carries-a-pose-per-view-where-openvr-collapses-to-one)
- [Dormant native stereo paths (check before you build)](#dormant-native-stereo-paths)
- [The clip-space stereo footer: stereo without finding the camera](#the-clip-space-stereo-footer-geometry-stereo-without-ever-finding-the-camera)
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
- [The switch you cannot find may be an argument, not a global](#the-switch-you-cannot-find-may-be-an-argument-not-a-global)
- [A repeated launch is not an ASLR test](#a-repeated-launch-is-not-an-aslr-test)
- [A third-party stereo fix is free intelligence about the engine](#a-third-party-stereo-fix-is-free-intelligence-about-the-engine--read-it-dont-install-it)
- [A proxy DLL must export everything the target actually imports](#a-proxy-dll-must-export-everything-the-target-actually-imports)
- [The instrument can be the bug](#the-instrument-can-be-the-bug)
- [Counting callers separates what a binary links from what it uses](#counting-callers-separates-what-a-binary-links-from-what-it-uses)
- [Both eyes from one recorded frame: resubmitting the command buffers](#both-eyes-from-one-recorded-frame-resubmitting-the-games-own-command-buffers)
- [D3D9 to a modern VR compositor: the shared-handle bridge](#d3d9-to-a-modern-vr-compositor-the-shared-handle-bridge-and-its-two-traps)
- [Never gate a state change on equality with a lerp target](#never-gate-a-state-change-on-exact-equality-with-a-value-that-only-lerps-toward-its-target)
- [When a game compiles its shaders decides how you read its constant map](#when-a-game-compiles-its-shaders-decides-how-you-read-its-constant-map)
- [The executable can name its own compressed formats and type hashes](#the-executable-can-name-its-own-compressed-formats-and-type-hashes)

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

## The clip-space stereo footer: geometry stereo without ever finding the camera

`[reported 2026-09-01]` — from NVIDIA's own published developer documentation. Nothing here has been
built or run by this account.

This library spends a lot of effort on [finding the camera matrix the engine actually
reads](#finding-the-camera-matrix-the-engine-actually-reads), because that is the usual way to two
eyes. **There is a second route that never touches the camera at all**, and it is the documented
mechanism behind NVIDIA 3D Vision Automatic — and therefore behind the geo-11 / HelixMod / 3Dmigoto
ecosystem this library already catalogues as [generic drivers](../generic-drivers/).

The driver monitored **vertex shader creation** and appended a footer to every shader. The footer
operates in **clip space**, chosen because it sits directly before the perspective divide, so a
horizontal shift there changes apparent stereoscopic depth **without moving the rasterised position
or the z-buffer depth** of the resulting fragments. The whole of it is one line:

```hlsl
ClipPos.x += Separation * (ClipPos.w - Convergence);
```

Each application draw call is then issued **twice**, with `Separation` positive for one eye and
negative for the other, into per-eye buffers.

**Read what that costs you: nothing about the application's camera, view matrix, projection matrix or
handedness needs to be known.** `w` is the view-space depth that every projection already produces,
and the two scalars are yours to choose. For a proxy that can already see draw calls and shader
creation, this is the cheapest known path from "I can see the rendering" to "I have two correct eyes"
— and it needs no NVIDIA driver, no NVIDIA GPU, and no surviving 3D Vision ecosystem. It is real
geometry stereo, not reprojection.

**What it does not give you is head tracking.** It is orthogonal to the 6DoF problem and solves none
of it — but it is combinable, and it means stereo need not be blocked on the camera hunt.

### The documented costs, which are the reason this needed per-game profiles

- **Not every draw should be stereoised.** Skyboxes, HUD, full-screen quads and anything already in
  screen space must be excluded, or they separate wrongly. NVIDIA used heuristics *plus a hand-built
  per-title profile from its own QA*. This is [main-camera
  discrimination](#main-camera-discrimination) arriving from a new direction, and it is precisely why
  3D Vision needed thousands of per-game profiles and why HelixMod fixes are per-game.
- **Post-processing and deferred renderers break.** Both *unproject* from window space back toward
  world space, and that unprojection cannot undo a clip-space shift it does not know was applied.
  NVIDIA's documented remedy was to publish the live `Separation` and `Convergence` into a small
  texture that the **application's own shaders** sample to invert the transform. A mod doing this
  itself must patch those shaders too, or accept broken screen-space effects — which is a substantial
  caveat on any modern deferred renderer.
- **Convergence is a comfort parameter with no on-screen representation**, so it needs tuning per
  title, by eye, in a headset.

### ⭐ Taking over the stereo parameters texture — the cost above, turned into a lever

`[reported 2026-09-01]` — read directly from NVIDIA's **own published** 3D Vision developer
documentation by this sweep, which is as strong as `reported` gets: first-party, not hearsay.
Surfaced by
[`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr), which found the
texture compiled into a shipped game's shaders and then found that its layout is documented.

The cost list above ends on a resigned note: in Automatic mode the driver publishes live separation
and convergence into "a small texture" that the application's own shaders sample to undo the clip-space
shift, and a mod would have to patch those shaders. **That reading undersells it badly, in two ways.**
The texture is NVIDIA's `StereoParmsTexture`, its layout is published, and — corrected 2026-09-02 from
an earlier draft of this section — **the application writes it itself; the driver never touches it.**
`ParamTextureManager` is a helper class shipped *in the `nvstereo.h` SDK header the application links*,
not a driver component: it calls ordinary NVAPI queries (`Stereo_GetSeparation` /
`Stereo_GetConvergence`) and writes the result with ordinary D3D resource calls. The driver's only
role is reading a signature back out of the finished texture to recognise it.

| channel of `pixel(0,0)` | contents (NVIDIA's own wording) |
| --- | --- |
| `.r` | *"Eye-specific separation"* |
| `.g` | *"Covergence"* — NVIDIA's own spelling, worth knowing when grepping |
| `.b` | *"Unit Vector identifying the current eye"* — **left eye = −1, right eye = +1** |

`[reported 2026-09-02, from NVIDIA's `nvstereo.h` as vendored unmodified in the open-source 3Dmigoto
project]` its **shape is fixed and known, not guessed at**: `StereoTexWidth × StereoTexHeight` is
**8 × 1**, `StereoTexFormat` is `D3DFMT_A32B32G32R32F` (D3D9) / `DXGI_FORMAT_R32G32B32A32_FLOAT`
(D3D10/11), and the texture carries an identifying marker,
`NVSTEREO_IMAGE_SIGNATURE` = `0x4433564E` ("NV3D"), written into it by the app itself.
`UpdateStereoTexture` is documented as called **once per frame, at the beginning of the frame, "even
while the device is lost."**

**Because the app writes this resource from values it queries itself, taking it over needs no race
against a driver thread** — hooking the app's own NVAPI calls or its texture-update/bind calls is
sufficient, and the exact byte layout can be predicted before a single capture: an 8×1 four-float
render target with a known signature is a resource-creation task, not a reverse-engineering one.

**Why this matters far beyond NVIDIA's ecosystem.** A `.b` channel that *tells a shader which eye it
is drawing* is a per-eye control channel sitting in an ordinary texture resource — and a proxy already
owns `SetTexture`. So on any title whose shaders sample this texture:

> **Bind your own texture with your own separation, convergence and eye sign, and every shader that
> samples it behaves per-eye — with no NVIDIA driver, no 3D Vision, no NVIDIA GPU, and not one shader
> patched.**

That is the division of labour NVIDIA designed, with **the mod in the driver's role**. Combined with a
located view-projection register it fully specifies a stereo implementation without a single launch:
render twice; per eye write that eye's view-projection; per eye bind a stereo texture whose `.b`
matches that eye's sign with matching `.r`/`.g`; leave every shipped shader exactly as it is.

**⚠️ One honest tension, recorded rather than smoothed over.** A `.b` channel that *informs* a shader
which eye it is in is the signature of the **Automatic** correction pattern — an application rendering
in Direct mode already knows which eye it is drawing and needs no texture to tell it. Yet Epic's own
integration of this into UE3 is titled for **Direct** mode (see the
[Unreal 1–3 engine page](../engines/unreal-1-3.md)). Both readings are live and the evidence genuinely
points both ways. **Fortunately the plan above does not depend on which is true**, because in either
case the texture is what the shaders read and the proxy is what binds it — and
[counting callers](#counting-callers-separates-what-a-binary-links-from-what-it-uses) settles the mode
statically on any specific title.

**How to find out whether a game has one, before launching anything:** the companion sampler and the
`NvStereoEnabled`-style branch constant are compiled into the shipped shader cache and are visible to
ordinary shader reflection. On one UE3 title the stereo branch constant appears in **65% of every
pixel shader in the game**. See
[read the shipped files before you attach anything](#read-the-shipped-files-before-you-attach-anything).

### ⚠️ The diagnostic that matters for recon: Automatic vs Direct

3D Vision had two modes, and confusing them will make you overrate a game:

| | **Automatic** | **Direct** (NVAPI) |
| --- | --- | --- |
| Who splits the draws | the **driver** | the **application** |
| Who owns separation/convergence | driver + per-game profile; the **user** tunes with `Ctrl+F3`/`Ctrl+F4` | the application |
| What the app must do | nothing — but its post-processing breaks unless it reads the stereo-parameters texture | render left, render right, present |
| How selected | default | `NvAPI_Stereo_SetDriverMode(..._DIRECT)`, **before device creation** |

**So a game whose binary references NVAPI stereo is not necessarily a game that renders two eyes**,
and a game with working `Ctrl+F3`/`Ctrl+F4` separation hotkeys is showing you *the driver's* controls,
not its own. In Automatic mode a title's stereo symbols are a **correction layer over work the driver
did**. Finding `Separation` / `Convergence` uniforms in a 2008–2013-era renderer is therefore evidence
of 3D-Vision *awareness*, not of a native two-eye path — and which driver mode the title requests
separates the two **statically**, before anything is launched.

**⚠️ "Statically" assumes `.text` is readable at rest — check that first.** `[measured 2026-09-02,
Alice: Madness Returns]` A wrapped executable (measured `.text` entropy at the theoretical ceiling,
entry point sitting outside `.text` in its own high-entropy section, zero `CC` padding runs — the
[packed-binary](#packedself-protecting-binaries) signature) makes the caller-count scan a guaranteed
false negative, not a real answer, because there is nothing to disassemble on disk. The gate this
re-triggers is `[PD]` → `[FLAT]`: the same method needs a **live memory dump of `.text`** first,
exactly the live-scan fix that section already prescribes. **A second trap on the same binary:** its
readable `.rdata`/`.data` can still carry a full NVAPI interface dispatch table (the linked SDK's
fixed list, one entry per function the SDK ships) with the two mode-selecting entries absent — do not
read that absence as "must be Automatic mode" before running the positive control: check whether
`NvAPI_Initialize` itself — which the game certainly calls — is also missing from the same table. If
it is, the table proves nothing about which functions the game *uses*, only which SDK it *linked*.

### It also names a mechanism the library had been treating as a black box

[Dormant native stereo paths](#dormant-native-stereo-paths) warns that vintage stereo may apply
separation as a projection/screen-space step rather than as two offset eye views. That caution is
right — and this section says the idiom is a **known, characterised technique of the era** rather than
a puzzle. (Context only: it is not a claim about any particular engine. id Tech 6's case is separately
and better resolved, where id's own published lineage genuinely does move `vieworg` per eye.)

Contributed by a `/gr` estate sweep, 2026-09-01, out of the `alan-wake-vr` 3D Vision question; the
technique itself is engine-agnostic.

## OpenXR carries a pose per view where OpenVR collapses to one

`[verified-static 2026-09-02]` from Khronos's own published `openxr.h`, following up a
`[hypothesis]` two sibling projects independently reasoned their way to hours apart on the same day.

**The problem this answers.** A same-frame stereo submission (needed by AER and by any true two-eye
render) wants to hand the runtime two independent poses, one per eye, in the same frame. On
**OpenVR**, [issue #1253](https://github.com/ValveSoftware/openvr/issues/1253) — filed by
**LukeRoss00**, the author of the AER technique this library already documents, describing exactly
this wall — has sat open for seven years with no Valve response: `IVRCompositor::Submit` is called
once per eye, and SteamVR keeps only the pose from the **last** call, discarding the first. A
same-frame two-eye submission over OpenVR therefore cannot carry two different poses at all.

**OpenXR's projection layer does not have the same shape.** Reading the SDK header directly rather
than reasoning about it: `XrCompositionLayerProjectionView` carries its **own `pose`** and its **own
`fov`**, and `XrCompositionLayerProjection` holds an **array** of those views (`viewCount` + `views`)
submitted **together, in one layer, in one space** — there is no separate per-eye submit call for the
last-one-wins collision to happen to. Per-eye poses are expressible in OpenXR **by construction**.

**⚠️ What this does and does not settle.** That the API can express independent per-view poses is now
a specification fact, not an inference. That a given **runtime** honours them independently during
reprojection is a separate, empirical question the specification cannot answer — and notably
SteamVR's OpenXR runtime shares a vendor with the OpenVR path that has the unfixed defect. So: the
design is no longer blocked at the API level, and the remaining risk moved from *"impossible"* to
*"untested per runtime,"* settled by one cheap headset test — submit two views with deliberately
different poses and confirm both are honoured rather than collapsed.

**A layer-type trap worth naming separately:** an OpenXR host built for a flat "one image, both eyes"
M1 milestone commonly uses `XrCompositionLayerQuad` — a flat rectangle with a **single** pose. That is
the right layer for M1 and the wrong one for true per-eye stereo; a quad-layer host proves the OpenXR
plumbing works, not that per-eye submission does. The **projection** layer above is the one this
section is about, and it is a different code path even though the swapchain/session handling is
shared.

Generalised from `far-cry-2-vr` (which opened the question against OpenVR's known defect) and
`XIII2003-vr` (which verified it against the header and found the layer-type gap in its own OpenXR
host), both same-day, 2026-09-02.

### ⚠️ Expressible is not honoured — two public reports, two runtimes, opposite directions

`[reported 2026-09-03]` Everything above is about what the **API** can express. Whether a **runtime**
acts on independent per-view poses is a separate question, and the public record answers it worse
than *"untested"*: it answers it **differently for different runtimes, and differently at different
times**.

- **2020-10-29 — SteamVR wrong, Oculus and Microsoft right.** **LukeRoss00** — the same developer
  who filed the OpenVR #1253 defect above — reported on Valve's own SteamVR discussion board that
  submitting the spec-correct per-view poses from `xrLocateViews` through `xrEndFrame` produced a
  **wrong stereo baseline and a vertical offset between the two eyes** on a Valve Index (SteamVR
  1.15.4, its OpenXR runtime 0.1.0). His workaround was to submit **the head pose for both views**
  and to swap the two views' `fov.angleDown` — i.e. to deliberately depart from the specification in
  order to get a correct picture. He recorded the Oculus and Microsoft runtimes as handling the same
  code correctly. The thread carries no reply.
- **2023-09 — the report inverts.** On the Khronos forums, **SirKandela** (Chaos LTD) reported the
  **Oculus desktop runtime appearing to ignore** the submitted `XrCompositionLayerProjectionView`
  pose, holding both projections centred on the HMD, and explicitly noted that **SteamVR respected
  it**. **Rylie Pavlik** replied that a runtime genuinely ignoring the pose would break timewarp
  outright — reprojection needs to know the pose an image was rendered for — so *"ignored"* may be
  the wrong description of what was observed. The reporter's own resolution was to abandon the
  projection layer for a quad layer.

**Read the pair, not either one.** They are three years apart and they name **opposite** culprits,
and that is the finding: per-view pose handling is **runtime- and version-specific, and it has
changed**. Neither *"it works"* nor *"it is broken"* can be written into a dossier as a general fact
about OpenXR, and any design that depends on independent per-view poses carries a per-runtime,
per-version risk that must be re-checked on the target rather than inherited from a sibling project.

**⚠️ It also changes how to test for it.** The obvious experiment — submit the two views with
deliberately opposite lateral offsets and watch for the image to split between the eyes — is weaker
than it looks, and the spec text says why: `pose` and `fov` *"should almost always derive from"* the
`XrView` values `xrLocateViews` returned, so a synthetic offset is precisely the off-the-beaten-path
submission a runtime is least likely to have been tuned for. A null result is then **ambiguous**
between *"this runtime collapses per-view poses"* and *"this runtime declined an implausible pose"* —
the [silent no-op](#silent-no-ops-verification-that-cannot-see-the-failure) shape this library keeps
running into.

**The stronger test is the one LukeRoss's report hands you for free: submit the legitimate located
per-view poses and look for his failure signature** — a visibly wrong stereo baseline together with a
**vertical** misalignment between the eyes. The vertical disparity is the valuable half: nothing in a
correct stereo pair produces it, so seeing it is a **positive identification of the defect** rather
than the absence of an expected effect. Run the synthetic-offset test too where it is already built —
it is cheap, and a clean split is genuinely informative — but do not let a null from it stand as the
answer.

**⚠️ Confidence, stated rather than implied.** Both accounts are first-hand developer reports on
forums, on hardware and runtime versions that are now years old, and neither has been reproduced
here. This moves the risk from *"untested"* to *"known to vary between runtimes"*; it does **not**
say what today's runtime does on your headset. That still costs one headset run.

Sources, read online: LukeRoss00's report on the
[SteamVR discussion board](https://steamcommunity.com/app/250820/discussions/8/3001046778344834329/)
(2020-10-29) · SirKandela's thread with Rylie Pavlik on the
[Khronos forums](https://community.khronos.org/t/oculus-runtime-ignores-projection-layer-views-pose/110078)
(September 2023) · the `XrCompositionLayerProjectionView` reference page in the
[Khronos OpenXR registry](https://registry.khronos.org/OpenXR/specs/1.1/man/html/XrCompositionLayerProjectionView.html).
Relevant right now to `XIII2003-vr` (which has a projection-layer submission path built and a
deliberate-offset test compiled in) and to `far-cry-2-vr` (blocked on the identical question for its
AER submission); a pointer went into both projects.

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

**A static entropy check finds this before you even try to disassemble.** A wrapped/encrypted `.text`
section reads at or near the theoretical entropy ceiling (**8.00**), its entry point commonly sits
**outside** `.text` in a separate high-entropy wrapper section, and it will show **zero `CC` (`int3`)
padding runs** — a tell no genuine MSVC-compiled code section produces, since compilers pad function
gaps with them. `[measured 2026-09-02, Alice: Madness Returns]` All three read instantly from the PE
headers and a byte histogram, with no disassembler needed, and they say plainly "there is nothing
real here to read yet" before a session spends time on a static scan that cannot possibly succeed.

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

**During a flat side-by-side stereo proof, leave the 2D layer full-window mono on purpose.** The mouse
maps to the whole window, so a menu squashed into one half or duplicated into both is unclickable, and
you lose the console you need to drive the test. Restore the full-window viewport before any 2D draw;
a HUD that appears in one half only is the restore logic missing a path, not a stereo bug. Moving the
HUD into the world is a later milestone. (Unreal Gold, M2 design decision, `[compile-verified 2026-09-02]`.)

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

### The residual may be a second gate: a PVS steps with position, a frustum sweeps with yaw

`[reported 2026-09-02]` The table above leaves a residual that the two levers never close, and it is
worth knowing that **the residual may not be the same problem at all.** Many engines run **two
visibility gates in series**, and only one of them follows the camera's orientation:

| gate | keys on | behaviour as you turn your head |
| --- | --- | --- |
| **frustum cull** | the camera basis — the matrix you are injecting into | varies **smoothly with yaw** |
| **from-region PVS / portal set** | which **leaf or room the camera is in** — a position, orientation-independent | **no change with yaw at all**; a **step change** when you cross into another leaf |

Psychonauts' level format turned out to ship a `VisibilityTree` separate from its collision tree and
its navmesh — an octree with one bit-buffer per leaf sized from `LeavesCount − 1`, i.e. **one bit per
other leaf: a precomputed from-region visible set**. Its per-object frustum test is a distinct
function taking a bounding box, which the yaw sweep above had already shown follows the camera basis.
Two gates, one matrix.

**The diagnostic is free and needs no headset.** Sweep yaw at a fixed position, then sweep position at
a fixed yaw, and score near-black pixels for both:

- black that **varies continuously with yaw** → frustum. Widening and camera-turning apply.
- black that is **flat across a full rotation but jumps when you move** → PVS. Widening the frustum
  cannot touch it, turning the engine camera cannot touch it, and **no amount of transform work will
  fix it** — you are outside what the level data says is visible from where you are standing.

**Two practical consequences, opposite in sign.** Moving the eye to the player's head for first person
is a **small translation that stays inside the same leaf**, so a PVS gate is unlikely to affect it. A
**flown free camera** is the opposite case: fly outside the level and the current leaf's visible set
can be empty, blacking the screen for reasons that have nothing to do with your matrices. Before
diagnosing a black free-camera frame as a basis or transform bug, check whether it **changes when you
rotate on the spot** — if it does not, stop debugging the matrix.

Generalised from [`psychonauts-vr/engine-research/`](https://github.com/TefMeister/psychonauts-vr/tree/main/engine-research)
(dossier §11, folded 2026-09-02 from a `/gr` pass over public work on the game's level format and its
open-source mod loader). `[reported]` rather than measured: the two-gate structure is read from public
documentation of the level format plus a located, partially-disassembled visibility function, and the
paired yaw-vs-position sweep described above has not yet been run.

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

`[verified-live 2026-09-01, n=5 projects]` (first seen 2026-08-26) The strongest single pattern this
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

### When a game compiles its shaders decides how you read its constant map

The recurring question — *which register or constant carries the view-projection, and what is it
called?* — is answered by a different technique depending on **when the game turns shader source
into bytecode**, not on which engine or graphics API it uses:

| When the game compiles | What ships on disk | How to read the constant map | Seen on |
| --- | --- | --- | --- |
| Ahead of time, source shipped | HLSL/`.usf` **source** | just read it — reserved-register comments are usually right there | Enslaved (UE3 ships `Common.usf`; `c0` = `ViewProjectionMatrix`) |
| Ahead of time, source stripped | compiled bytecode **with a reflection block** | parse it — D3D9 bytecode carries `CTAB` naming every constant/register, D3D10+ carries `RDEF` | Alice: Madness Returns (45,832 `CTAB` tables), Enslaved (34,046), Mad Max (`RDEF`) |
| At runtime | HLSL plus a shipped **shader-compiler redistributable** | **hook the compiler** — proxy `d3dcompiler_4x.dll` and log every source string, entry point and define as it compiles | Alan Wake (ships `d3dcompiler_42`/`43` cabs; fails with *"could not process hlsl shader"* without them) `[inferred-static 2026-09-02, from the redistributable + the error string]` |

**The tell for the third case is in the install folder, not the binary.** A shipped shader-compiler
redistributable, or a startup error naming HLSL, means the bytecode does not exist until load — so a
session that goes looking for a shader cache will correctly find nothing on disk and can wrongly
conclude the game ships no shaders to read at all. Alan Wake is the worked case: the two UE3 siblings
above answered the same question by parsing `CTAB` off a compiled cache, and that trick does not
transfer here because there is no cache to parse — the game reaches `D3DCompile`/`D3DXCompileShader`
in `d3dcompiler_4x.dll` **by name, from a DLL sitting in its own install folder**, the same shape as
proxying `nvapi.dll` for a stereo call. A proxy on that one export sees every shader's source text,
entry point and defines as they compile, in one run.

Two things worth stating about the runtime-compile case specifically: it is the **easiest** of the
three to read, not the hardest — the source names its own constants in plain text, and the compile
call is one chokepoint in one DLL, so a single proxied export yields the whole corpus with names,
where the compiled-cache case yields only registers and the shipped-source case depends on the
developer having shipped sources at all. And it is the only one of the three that is **upstream of
the bytecode**, making it the one place a per-eye term could eventually be added without patching
bytecode or overriding a constant post hoc — a large commitment, and not a first move.

**The general habit:** before planning any capture, check what the install folder says about *when*
shaders become bytecode. A `d3dcompiler_*.dll` or a compiler cab sitting in the game's own tree is as
informative as the renderer's import table, and it changes which of the three techniques even
applies. Generalised across `enslaved-vr`, `alice-madness-returns-vr`, `mad-max-vr` and
`alan-wake-vr`, 2026-09-02.

### The executable can name its own compressed formats and type hashes

Two related tricks for reading an unknown packed/serialized format statically, both cheaper than
inferring the format from byte patterns:

- **Compression algorithm identity is often sitting in the strings table.** Before guessing an LZ
  variant from byte patterns, grep the executable for the compression library's own **enum
  strings** — one project carried `LZO1X_1`, `LZO1X_999`, `LZO2A`, `LZX` as a contiguous run, and a
  chunk header's small integer type field picked one straight off that list. Transcribing the
  matching public decoder's published constants then reproduced every compressed block to the exact
  output size with exact input consumption — a stronger check than any checksum, and available
  *before* the checksum algorithm is even known. `[verified-numerically 2026-09-02, 7.90 GB
  reproduced]`
- **A type/identifier hash stored beside a name is, more often than not, plain CRC32 of that name.**
  Before assuming a custom hash function, check `crc32(candidate_name)` against any 32-bit value the
  binary stores alongside a name (a type registry, a reflection table). Once confirmed, a CRC32
  dictionary built from every identifier-shaped string in the executable can resolve a large fraction
  of otherwise-opaque stored hashes into named, typed objects in one pass — one project resolved 201
  of 202 this way. Applies to any engine with reflection-style or registry-style type tables, not
  just the one it was found on.

Generalised from `prince-of-persia-2008-vr`'s static `.forge`/Scimitar decoding session, 2026-09-02
(no launch); engine-specific layout detail stays in that project's own dossier.

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

`[verified-numerically 2026-09-01, n=1 game]` (Dunia / Far Cry 2) When a head-tracking composition comes out
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

### Measuring eye height for a first-person conversion: camera-minus-player is a *camera* height

`[measured 2026-09-02, Psychonauts]` A third-person game's camera position minus its player position is
the height of the **third-person camera**, not of the character's eyes — in the worked case it was
about 328 units by one camera reading and about 149 by another, and neither is an eye height. Recording
either as one would have baked a wrong constant into the first-person work at exactly the point where
the default was already flagged as a guess. Two rules:

- **Take eye height from the skeleton, not the camera:** the head bone's world position minus the
  player's root position, both in the engine's own units, with no camera in the path and no unit-scale
  conversion. Most engines expose a bone-world-position call; find it before measuring anything.
- **Check the ground is level where you measure.** The player's up-axis value drifted by tens of units
  across a short walk in what looked like a flat car park, so a single sample there would have been
  wrong twice over.

From notes 71–72 in [`psychonauts-vr/modding-notes/`](https://github.com/TefMeister/psychonauts-vr/tree/main/modding-notes).

## Silent no-ops: verification that cannot see the failure

Four independent cases, two of them in major public tools, one our own and one documented outright
by the vendor, all share a shape worth naming: **a check that appears to confirm success while being
structurally incapable of detecting the failure.** None of them threw, logged an error, or left a wrong value
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
- **The worst variant: an API that offers no verification surface at all, by design.** `[reported
  2026-09-01]` Microsoft's own reference for
  [`ID3D11DeviceContext::ExecuteCommandList`](https://learn.microsoft.com/en-us/windows/desktop/api/D3D11/nf-d3d11-id3d11devicecontext-executecommandlist)
  states that the runtime validates around queries, and that where a query begun on one context
  would be manipulated indirectly by the list, **the method does not execute the command list** —
  while still clearing the context state on its way past. **The method returns `void`.** There is no
  `HRESULT`, no out-parameter, nothing to check: an entire list can be submitted, discarded, and
  take the context state with it, leaving a frame that renders wrongly with nothing raised anywhere.
  For a VR hook that injects, reorders or duplicates command-list execution around a renderer using
  occlusion queries, timestamps or predication — all ordinary at this class — the symptom is a
  missing or wrongly-stated pass, and the natural misdiagnosis is *"my patch is wrong"* rather than
  *"my list never ran"*.
  - The guard is cheap and specific: **bring up with the D3D11 debug layer enabled.** The validation
    quoted above is exactly what the debug layer surfaces, so it turns an invisible discard into a
    message — worth doing *before* chasing a patching bug, not after.
  - A quieter hazard sits on the same call. Its `RestoreContextState` parameter, passed **`FALSE`**,
    returns the target context to its **default state** after execution — and the documentation
    recommends `FALSE` for performance
    ([Command List, Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3d11/overviews-direct3d-11-render-multi-thread-command-list)).
    So on most real engines anything running afterwards inherits **nothing**: no render targets, no
    constant buffers, no shaders. A per-eye pass written to assume it inherits what the previous
    list left will fail in a way that reads as a patching bug rather than a state-management one.

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
- **When a function returns `void`, find out what it does on failure before you rely on it.** A call
  that *cannot* report failure has not thereby become infallible — it has moved the failure report
  somewhere you have to go looking for it. Where a debug or validation layer exists, that is
  normally where it went, which makes "enable the validation layer during bring-up" a structural
  precaution rather than a debugging step. This generalises past D3D11 to Vulkan's validation layers
  and to any `void` submit/execute entry point.

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

**And before any of that, read the game's bindings.** `[disproved 2026-09-02]` A keystroke delivered
perfectly to a key the game does not bind is indistinguishable, from outside, from a keystroke that
never arrived. Psychonauts moves on the arrow keys; two seconds of flawlessly injected `W` proved
nothing and was very nearly recorded as an input blocker (worked example under
[controls, rule 1](#1-before-recording-a-negative-as-fact-confirm-the-test-could-have-gone-positive)).
Send a key the game is known to act on, and when the repo already has a helper that once moved the
player, use it before writing a new one.

### Known input routes, by engine family

Compiled from our own live tests; incomplete on purpose, extend it as you measure.

| Engine family | `SendInput` | Posted window messages | Notes |
| --- | --- | --- | --- |
| Unreal Engine 2 era | **No** — exclusive DirectInput for the mouse | keyboard works | Mouse and keyboard take different routes; test them separately |
| Capcom RE Engine | **No** | **Yes** (`WM_KEYDOWN`/`WM_KEYUP`) | `[verified-live 2026-08-24]` |
| Double Fine bespoke (Psychonauts) | mouse: **no** | keyboard reaches gameplay, **not** menus | Title/credits screens need a real gamepad · movement is on the **arrow keys** (DIK scancodes, extended flag), not WASD — an unbound `W` produced a false negative on 2026-09-02, re-verified `[verified-live 2026-09-02, n=2 directions]` |
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

**A fourth shape, from a second project** `[disproved 2026-09-02, Psychonauts]` — **a perfectly
delivered stimulus the system was never bound to respond to.** Three keyboard/mouse injections
produced no movement and no view rotation, with the window confirmed foreground and the frame counter
confirmed live, and the negative was written up with two candidate causes — one of them the project's
*own* earlier bug-fix, complete with a plausible mechanism. The keystroke had been delivered flawlessly
to a key the game does not bind (movement is on the arrow keys, not `W`), and the arrow that was tried
used a different scancode encoding from the one the repo's proven helper uses. The positive control — a
helper that had demonstrably walked the player days earlier — was in the same repository and was not
consulted. Two lessons compound. **Look for the positive control you already own before theorising**:
had nobody checked, a future session would have hunted a defect in working, load-bearing code. And
**three parameter sets of one API are not three routes** — `SendInput` scancode, `SendInput` extended
scancode and `SendInput` mouse were never independent, and mistaking them for a spread of routes is
what made the null look like a wall. Notes 71 and 72 in
[`psychonauts-vr/modding-notes/`](https://github.com/TefMeister/psychonauts-vr/tree/main/modding-notes),
the second superseding the first.

**A fifth shape, from a third project** `[verified-numerically 2026-09-02, Prince of Persia 2008]` —
**a negative search result would have been load-bearing, so a positive control was run first.** The
plan was "search a game's serialized state data for the hash of a known, certainly-reachable state
name" (to prove state identifiers are stored as hashes at all, before hunting for one specific
target state). Three control hashes — states that indisputably run in normal play — matched only
inside audio data, never inside the state records themselves. That told a different and more useful
story than a null on the target ever could have: **the data does not store states as hashes** (it
uses plain ordinals), so the target's absence-as-a-hash was never evidence it was missing — it was
evidence the search category was wrong. Skipping the control would have recorded "the debug camera
was stripped from the shipping build," and it was in fact present and authored. **Generalises beyond
this one case: whenever a negative on a specific target would be recorded as a finding, first run the
identical search for something that certainly must be there — if that also comes back empty, the
method is wrong, not the target.**

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
7. **Check your signal can separate the states before you threshold it.** Distinct from guard 5:
   there the two conditions were accidentally the same; here they genuinely differ and the *chosen
   measurement* cannot tell them apart. Working the RE2 arcade-controls front, camera-to-head
   distance was proposed as a "has the camera left first person?" test and measured **0.111 m at
   rest against 0.112 m during an actual enemy grab** — the camera never leaves the head, so **no
   threshold could ever have worked**, and any tuning effort would have gone into an undecidable
   question. The same project's other attempt failed the opposite way: a damage flag was true for
   *any* hit, including an ordinary punch that never leaves first person — a signal firing far too
   widely. **Before tuning a threshold, measure the candidate signal in both states you need to
   distinguish and look at the gap.** If it is within noise, or if one state is a superset of the
   other, change the signal rather than the threshold. `[measured 2026-08-24, n=2 signals]`

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

### And check whether the *community* already built it — then check it does not collide with your proxy

`[reported 2026-09-01, n=1 project]` From
[`psychonauts-vr`](https://github.com/TefMeister/psychonauts-vr). The two habits above ask whether the
*game* shipped the thing you are about to build. This is the adjacent question, and it caught a
primitive that had been recorded as unfinished for weeks: **before you finish building it, check
whether someone else already did.**

The find was a community mod loader for the exact modern release being modded, shipping an in-game Lua
console, restoration of the game's own native debug menu and level select, restored debug rendering,
and widescreen support. Three of those map one-to-one onto items this account's own dossier carried as
*unfinished* or *live test in progress* — including a debug-menu question that had been open for a
week. Whatever such a tool does is at minimum a **cross-check on your own approach**, and at best the
answer.

Two things generalise:

- **Search the ecosystem, not just the tool.** The signal that the community's programmatic reach into
  a game is deeper than you recorded is often not the loader itself but **what is built on it** — a
  randomiser, or a multi-game-randomiser integration, both of which necessarily read and write live
  game state and hook game events. If those exist, someone has already solved state access.
- **Look off GitHub.** This one is hosted on GitLab. A GitHub-only search would have concluded the
  game had no mod loader.

**⚠️ And then assess the collision, because a loader is an injector too.** A mod loader that patches a
game is doing the same kind of thing your proxy does, and the two can want the same slot — the same
proxied system DLL filename, the same import to hijack, the same hook site. Establish **which
mechanism it uses before installing it**, because a tool that helps in principle can silently break
the injection route your whole project depends on. Not installing it yet is a legitimate answer, and a
cheaper one than debugging the interaction later. Compare
[the instrument can be the bug](#the-instrument-can-be-the-bug) — that is this hazard arriving from
inside your own toolchain.


### Check whether the game shipped the comfort switch you are about to write code for

`[reported 2026-09-02, n=2 games]` Automatic camera motion — tilt, bob, shake, auto-centring, chase-cam
lag — is a first-order **comfort hazard** in VR and one of the first things a conversion wants gone. It
is also exactly the sort of behaviour a shipped game often exposes an off-switch for, because it
annoyed somebody on the development team too.

- Enslaved's chase-camera ini carries a `useAutoTiltup` flag that can simply be turned off.
- Alan Wake ships a `-rigidcamera` command-line switch of the same character.

Both were found by reading the game's own configuration and launch options — no hooking, no patching,
nothing to maintain across a game update, and no risk of fighting the engine for a value it will
rewrite next frame. Before writing a hook to suppress a camera motion, spend the few minutes on the
game's ini files, its launch arguments and its own gameplay options. The same reflex applies to motion
blur, depth of field and film grain, which
[have to be off anyway](#turn-off-the-post-processes-that-re-derive-the-view-before-judging-a-stereo-run)
before a stereo run can be judged at all.

Generalised from [`enslaved-vr`](https://github.com/TefMeister/enslaved-vr) and
[`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr).

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

- **A client-side-rendered project page versus an automated fetch.** `[verified-live 2026-09-01,
  n=2 sessions]` GitLab (and any site whose project/wiki pages render in the browser via JavaScript)
  returns only a loading skeleton to a plain automated fetch of the page URL — indistinguishable from
  a genuinely empty page. This produced a "licence and install method unread — needs a browser"
  conclusion that was really a tooling gap, not an absence of information. The fix is the same shape
  as the other rows here: **query the API that serves the real content instead of the page that
  wraps it.** GitLab's REST API is plain JSON/raw bytes and needs no token for public projects — a
  repository tree (`/api/v4/projects/<id>/repository/tree?path=<dir>&recursive=true`), a raw file
  (`/api/v4/projects/<id>/repository/files/<url-encoded-path>/raw?ref=<branch>`), or a wiki page
  (`/api/v4/projects/<id>/wikis/<slug>`) all return real content where the page fetch returns a
  shell. The numeric project id is on the project's front page. Same discipline as the `strings -n 4`
  row above: it was the tool, not the source, that produced the negative.

**A documentation host can 403 an automated fetcher while serving browsers normally.** Hit while
checking a specification page during the 2026-09-03 sweep: the Khronos registry returned
`403 Forbidden` to an automated fetch of a reference page that is publicly readable in a browser and
whose text a search engine had already indexed. The failure carries no hint that it is about the
*client* rather than the *content*, so it is very easy to record as "the page is gone" or "the spec
does not say". Treat a 403 from a docs, registry or wiki host as **"read it another way"** — a
browser, a search engine's cached text, a mirror, or the project's own source repository — and never
as evidence about what the document contains. Same family as the client-side-rendered page above: the
tool's limitation gets read as the world's.

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

## The switch you cannot find may be an argument, not a global

When a renderer clearly *has* a capability — stereo, a debug view, an alternate projection — the
instinct is to hunt for the thing that turns it on: a cvar, a config key, a mode enum, a global to
flip. That hunt can fail for a reason that looks nothing like failure: **the feature may never have
had an on-switch, because the mode is threaded through the render call as a parameter.**

DOOM 2016's dormant stereo path is the worked example, and it took **two independent negative reads**
to settle. The engine's **published cvar dump — all 6,572 of them, the binary's full inventory — was
read end to end** `[reported 2026-09-01]`: all four `stereoRender_*` parameters are in it, so are
`multiView_60Hz` and `com_production`, and **nothing in it selects the render mode**. Separately, the
retail build registers only **171** of those at runtime and `listCvars stereo` returns nothing
`[verified-live 2026-08-26]`. Two different questions — *does the name exist?* and *is it reachable
live?* — both answered no for a mode selector. An earlier note had recorded that the mode cvar's name
simply "was not resolvable statically, find it live"; that advice pointed at nothing, and is now
tagged `[disproved 2026-09-01]` in the project's own dossier rather than deleted, precisely because
it would otherwise have cost a live session.

**Keep those two reads apart when you do this yourself.** A name appearing in a shipped cvar dump,
symbol table or strings pass proves only that the *binary* knows it. Whether the running build
**registers** it is a separate measurement, and conflating the two turns "present" into "available"
— a mistake that plans live sessions around switches the process will never accept.

What the previous generation's **published** source shows instead `[reported 2026-09-01, from id
Software's GPL release of Doom 3 BFG]` is a call signature:

```c
void RB_DrawView( const void *data, const int stereoEye );   // 0 = mono, -1 / +1 = eyes
```

— with the eye carried downstream as first-class state on the view object (`viewEyeBuffer`: `-1`
left, `+1` right, `0` for mono or a GUI), and the per-eye GUI shift computed as
`stereoEye * stereoScreenSeparation`. The one cvar that *does* look like a switch,
`stereoRender_swapEyes`, turns out to be consulted only when comparing a shader's eye against the
current one — a late cosmetic flip, not the gate.

**The diagnostic is the shape of the inventory, and it is worth learning to read.** Every *parameter*
of a feature exposed as tunable state, while nothing selects the *mode*, is exactly what a call-site
argument looks like from the outside. It is not evidence that the feature was stripped, and it is
not evidence that the name is hidden. It is evidence that you are looking for the wrong kind of
object.

**So change what you hunt for.** Stop looking for a global to flip and start looking for **a function
that takes a small signed or enumerated argument and is called more than once per frame**, plus **a
matching field on the view/frame object**. Two practical consequences:

- **A named-field search beats a live cvar dump for this**, and costs no launch at all. Where an
  engine ships a reflection or symbol table (see
  [read the shipped files](#read-the-shipped-files-before-you-attach-anything)), searching it for an
  eye/mode field on the view struct is static work available immediately. Engines name things
  consistently across generations, so the ancestor's field name is a good query.
- **It re-prices whatever you were doing to reach the switch.** DOOM's console gate was being pursued
  partly to reach the stereo toggle; once the toggle is known to be an argument, opening the console
  yields the *parameters* and not the on-switch, which moves the whole gate off the critical path.
  A finding that removes work is worth as much as one that adds a lever.

`[reported 2026-09-01]` for id Tech 4/5, where the source is published; **`[hypothesis]` for id Tech
6**, a generation and several years later. What lifts the id Tech 6 case above a guess is that the
call-argument model *explains the observed cvar inventory* — it predicts exactly the pattern that was
measured. Generalised from
[`doom-2016-vr/engine-research/`](https://github.com/TefMeister/doom-2016-vr/tree/main/engine-research);
the underlying research came via a `/gr` pass.

## A repeated launch is not an ASLR test

A specific trap, cheap to fall into and cheap to avoid, recorded because this account fell into it
and had to withdraw the claim a few hours later.

Having found a camera or state address at some absolute location, the natural next question is
whether it survives a restart. DOOM 2016's module loaded at **the same base on three consecutive
launches**, and the conclusion drawn was that Windows randomises image base **per boot**, so only a
**reboot** could test rebasing.

**That is false** `[disproved 2026-09-01]`. A fourth launch, with **no reboot at all**, loaded at a
different base. The likely mechanism `[hypothesis]` is that the relocated image is kept while its
section object stays alive in the standby cache, and a new base is picked once it is evicted — which
is exactly why three launches in quick succession look pinned and one hours later does not.

**The methodological point is the transferable part, and it is not about ASLR.** Three trials that
all ran inside the same cache-warm window are not three independent trials; they are one trial
repeated. `n=3` counts only if the runs can actually differ from each other, and for anything
timing- or cache-dependent, *back-to-back* is precisely the arrangement that guarantees they cannot.
When a result is suspiciously stable, ask what the repeats had in common before recording the
stability as a property of the system. Space the runs, or vary the thing you think might matter,
before writing `n=K`. See also
[a negative needs a positive control](#controls-a-negative-needs-a-positive-one-a-positive-needs-a-no-op-one).

**The operational rule, meanwhile, never depended on the answer**, and that is the reassuring half:
resolve the address as `GetModuleHandle(NULL) + <RVA>` (or the target process's module base) every
session and re-verify before writing. That is correct whether the base moves per boot, per launch or
never, costs nothing, and is why the wrong belief blocked no actual work. **When a cheap procedure is
immune to an open question, adopt the procedure and stop needing the answer** — and note that the
question this leaves genuinely open is a different one: whether the **RVA** holds across a rebase,
which the same re-verification answers on the next run.

Generalised from
[`doom-2016-vr/engine-research/`](https://github.com/TefMeister/doom-2016-vr/tree/main/engine-research).

## A third-party stereo fix is free intelligence about the engine — read it, don't install it

Many older D3D9/D3D11 titles have a **stereoscopic-3D fix** published by the HelixMod / geo-11
community: a per-game shader patch that makes the title behave under a generic stereo driver. Our
projects keep finding that such a fix is worth reading closely **even when it is useless as a
component** — closed-source, wrong renderer, or simply not something we would ever copy. It is a
report written by somebody who already did per-eye work on this exact binary, and it answers
questions that otherwise cost live sessions.

`[reported 2026-08-25, n=4 projects]` — Alice: Madness Returns, Alan Wake, Prince of Persia (2008)
and Burnout Paradise. Three distinct kinds of intelligence come out of it.

**1. What the fix did *not* have to touch tells you what the game already gets right.** This is the
most valuable and the least obvious. The Alice: Madness Returns fix describes its own job as pushing
*"2D UI to 3D depths"*, and its author notes the game *"comes with Stereoscopic support"* that merely
"wasn't 100%". A fix scoped to the UI layer, on a game shipping its own stereo mode, is evidence that
**the native per-eye camera and projection path was already substantially correct** — the hard part
of the problem this library exists to solve. Alan Wake looked like the same signal from a different
direction — reported "almost 3D Vision ready out of the box", with separation adjustable in-game on
`Ctrl+F3`/`Ctrl+F4` — but **that one needs qualifying, and it is a useful correction to hold onto.**
`Ctrl+F3`/`Ctrl+F4` are the **driver's** hotkeys in 3D Vision *Automatic* mode, where the driver
splits the draws and owns the parameters; they are evidence that 3D Vision worked *on* the game, not
that the game contains a native per-eye path. See [the clip-space stereo
footer](#the-clip-space-stereo-footer-geometry-stereo-without-ever-finding-the-camera) for the
distinction and for the static check that settles it. **Alice's signal survives this and Alan Wake's
weakens**, because Alice's rests on the fix author's own statement that the *game* ships stereo
support, not on driver-side controls.

Where the signal does hold, the cheap next step is to find and toggle the native mode and watch what
changes in the constant registers between mono and stereo — far more direct than reverse-engineering
the mono path alone.

**2. The fix's own issue list is a free pass inventory.** The Prince of Persia (2008) fix enumerates
what broke under stereo: skybox depth (**and separately for the dark and sunny weather variants**),
lens/sun-flare doubling, UI rendered flat at screen depth, background-landscape depth — plus a
residual flicker on some effects that even the mature 2016 rebuild never fully fixed. That is a
ready-made list of the passes a from-scratch VR conversion will have to handle on the same engine,
compiled by somebody who hit each one, and it arrives before you have opened a capture.

**3. The fix's *configuration structure* encodes engine structure.** The same fix ships **separate
convergence presets for cutscenes and for exploration gameplay**. You cannot need two presets unless
the game drives the camera through two different paths. That is a structural fact about the engine
inferred for free from a settings file — and it says: scope the live investigation to check both
paths early, rather than assuming the one you found covers cinematics too.

### The caveats, which matter as much as the method

- **Check the fix targets your exact build and renderer, not your title.** Burnout Paradise is the
  worked negative: a HelixMod fix exists for the **original 2008 D3D9 release**, and the same source
  confirms **no equivalent exists for the Remastered D3D11 build**, which is the actual target. Name
  matching is worth nothing; the renderer and build have to match too.
- **A fix's existence proves the renderer is hookable, not that the conversion is tractable.** For
  Prince of Persia the fix is the project's evidence that D3D9-level hooking works against that
  binary — a genuinely weaker claim than a vorpX Geometry-3D profile, and much weaker than 6DoF.
  Keep the classes apart: shader-level stereo fix < generic-driver geometry stereo < engine-level
  6DoF.
- **A second iteration of a fix is itself a signal.** The Prince of Persia fix was rebuilt in 2016
  specifically because the 2012 version's blanket shader matching had broken unrelated combat
  effects; the rebuild had to distinguish shader/texture **pairs** to avoid it. That is a warning
  about the technique, not just that game: matching shaders too coarsely causes collateral damage.
- **Read online; take nothing.** These fixes are closed-source or unlicensed. Everything above comes
  from a fix's public description, changelog and settings documentation — no code, and no
  installation. See [`CONTRIBUTING.md`](../../CONTRIBUTING.md).

Generalised from [`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr),
[`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr),
[`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr) and
[`burnout-paradise-vr`](https://github.com/TefMeister/burnout-paradise-vr).

### Two additions from the 2026-09-01 sweep

**4. The announcement is not the intelligence — the fix's own files are.** A fix's blog post or
release page is written for players: hotkeys, autoconvergence, depth cycling, a list of what now looks
right. Checked directly on one such post, it **named no constant-buffer slot, no register and no
shader hash**. The register-level detail, where it exists publicly at all, is in the fix package's own
`.ini` and shader-assembly files in the community archive. Read those *online, in a browser*, and read
them for **facts about the game you own** — *"the shared constant buffer's float4 slot N is the
view-projection"* is a fact about that game, and it cross-checks against your own reflection dump in
minutes. Nothing may be copied; the implementation must be your own. This is the difference between
learning where a game keeps its matrices and taking someone's fix, and it is a real difference.

This matters most exactly where reflection runs out. When a shared per-frame buffer's type record
shows a raw `float4[N]` array with **no recoverable member names** — the engine fills it from C++, so
there is nothing to name — finding the view matrix means finding it *by value*, which costs a live
run, a capture and a pile of inference. A published fix may have written the answer down years ago.

**5. Check that the prior art you recorded is still the current prior art.** One project's dossier
carried a 2015 fix as its stereo precedent; the actual current state was a **2024 revision by a
different author, built on the modern geo-11 stack, targeting the last shipped game version**, with a
completely different and much longer list of fixed passes. Two things change when you notice: the
feasibility signal is far stronger than a nine-year-old artifact suggests, and **the newer fix's
per-pass list is a ready-made inventory of exactly which passes break under stereo in that renderer**.
Prior art has a date; re-check it before quoting it.


### A sixth addition, from the 2026-09-03 sweep

**6. "No fix exists" is a claim with a date on it, and it can simply be wrong.** One project's
external research recorded, on 2026-08-24, that no HelixMod or 3DMigoto entry existed for its game.
It was wrong: **eqzitara** had shipped one for that exact title in October 2013. The cost of the
error was a fortnight of treating the project as having **no** per-eye prior art when it in fact had
a decade-old report on the same binary. A negative about the public record is the easiest kind of
claim to get wrong — it depends on how you searched, not on what exists — so tag it `[reported]` at
best, put a date on it, and re-run the search before letting it steer a design. Compare item 5 above:
prior art can be **newer** than you recorded as well as **present** when you recorded none.

**And when the fix is found, its value is corroboration as much as inventory.** In this case the
project had already predicted, from its own shader-reflection work, that its vertex-constant
injection would leave a large family of pixel-stage passes uncorrected. The public fix's own list of
what it had to correct — shadows, crosshairs, effects, menus, with HUD depth still imperfect —
matched that prediction, arrived at by a different person, a different method and thirteen years
earlier. Two independent routes to the same pass list is a much stronger position than either alone,
and it promotes *"watch for anything odd"* into a ranked list with shadows at the top.

## A proxy DLL must export everything the target actually imports

The proxy-DLL foothold — drop a same-named `d3d9.dll` / `dinput8.dll` / `winmm.dll` beside the exe,
forward the real calls, intercept the interesting one — is this account's default way in, and it has
one failure mode that costs a session every time it is met fresh. **The failure looks like the game
rejecting your mod. It is actually the Windows loader rejecting your export table.**

`[verified-live 2026-08-25, n=3 projects]` The instructive contrast is between two games proxied in
the same week:

- **Alice: Madness Returns** statically imports **two** functions from `d3d9.dll` —
  `Direct3DCreate9` **and `D3DPERF_SetOptions`**, a real performance-marker export. A proxy exporting
  only `Direct3DCreate9` left the loader unable to resolve the executable's import table at all, so
  **the process died before running a single instruction**: about two seconds, no window, no log
  output, nothing written anywhere. Adding the second forwarding wrapper fixed it outright.
- **Prince of Persia (2008)** needed only `Direct3DCreate9`, and worked on the first attempt.

So the count is per-game, and **the DLL name appearing in the import table tells you nothing about
which functions are needed.** Enumerate the executable's actual per-function imports for that DLL
before writing the proxy — one `dumpbin /imports`, `objdump -x` or equivalent — and it is the
difference between a first-attempt success and an evening spent on a game that "just exits".

### The failure mode depends on *how* the game resolves the function

This is the part worth internalising, because the two cases need completely different debugging:

| Resolution | A missing export produces |
| --- | --- |
| **Static import** (name in the PE import table) | The loader fails the whole process **before `main`**. No log, no window, no error dialog, no crash report — an instant silent exit that reads as an incompatibility. |
| **Dynamic** (`LoadLibrary` + `GetProcAddress`) | The call site's own failure path runs — typically a logged, graceful error, because the developers wrote a handler for it. |

Alan Wake is the dynamic case: its `d3d_sf_Win32.dll` carries `Direct3DCreate9` and
`Direct3DCreate failed` as adjacent strings, the signature of a `LoadLibrary` + `GetProcAddress` pair
with a handled failure. A same-named proxy still works — `LoadLibrary` follows the same
application-directory-first search order — but an incomplete one degrades into the game's own error
message instead of killing the process. **If your proxy produces total silence, suspect the export
table; if it produces a tidy error message, suspect your logic.**

**A related check that pays for itself:** confirm how many functions of that DLL the game references
*anywhere*, not just the one you plan to intercept. Alan Wake's recon verified `Direct3DCreate9` was
the only D3D9 export referenced across all ten of its module binaries — a check performed
specifically because of the Alice result. That is the re-audit habit working as intended.

## The instrument can be the bug

`[verified-live 2026-08-25, n=1]` A single well-documented case, recorded because the shape is
general and the debugging cost was real.

A from-scratch `d3d9.dll` proxy crashed Alan Wake outright — an access violation in `ntdll`, with a
control test (proxy removed) launching fine. The natural response is to instrument: an
`IDirect3D9::CreateDevice` vtable hook was added **to see further into the failure**. The game then
failed reliably. A Windows Fault-Tolerant Heap compatibility shim was tried next, on the theory that
a latent 2010-era heap bug was being exposed, and briefly appeared to help.

**Both readings were wrong. The vtable hook was itself the cause.** With the hook disabled and
nothing else changed, the game launched and ran cleanly; removing the FTH shim afterwards changed
nothing either way. The instrument added to observe the fault was manufacturing it, and the shim had
drawn credit for a fix it never performed.

Three transferable habits:

- **When a diagnostic is added and the failure changes, the diagnostic is a suspect — not just a
  lens.** Hooks, logging wrappers and interposed vtables are code running inside someone else's
  process at a moment the process is fragile. Test with the instrument disabled and everything else
  identical, the same way you would test with your mod removed. This is
  [remove your own code before accepting the blame](#remove-your-own-code-before-accepting-the-blame--then-fix-the-producer)
  applied one level in: your *debugging apparatus* is also your own code.
- **Never let a mitigation take credit while another variable is moving.** The shim "seemed to help"
  because it was applied in the same window as other changes. A mitigation earns belief only from a
  run where it is the single difference — and the cheap confirmation is to **remove it again
  afterwards** and check the fix survives. Here it did, which is exactly how the red herring was
  caught.
- **Keep a known-bad instrument, disabled, with a note.** The hook was left in the proxy source,
  switched off, with a written warning not to re-enable it before understanding why it broke startup.
  That preserves the finding for whoever needs `CreateDevice` interception later, instead of leaving
  a deleted mystery for them to rediscover.

Generalised from [`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr).

## Counting callers separates what a binary *links* from what it *uses*

`[inferred-static 2026-09-01, n=1 game]` — read out of the binary, never seen running; the
method was reproduced here against a first-party ID table.
Generalised out of [`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr), whose §6 was decided
by it without the game ever being launched.

A recurring recon question has the shape *"the binary references API X — does it actually **use** it?"*
It matters because the two answers point at opposite projects. Alan Wake's renderer references NVAPI's
stereo functions; if it drives them it has a self-rendered two-eye path and a VR mod inherits half its
work, and if it merely links them it is a **correction layer over a driver that no longer ships** and
the camera must be found the ordinary way. Same symbols, opposite conclusions.

**The trap is that "the symbol is present" answers nothing.** Unused dispatch stubs get linked in.
Import tables list what the loader must resolve, not what the code path reaches. A string search or an
import dump will report a stereo-capable binary either way.

### The method

1. **Find each wrapper's call site, not its name.** Many C APIs — NVAPI is the clean example — resolve
   their entry points through a single `QueryInterface`-style dispatcher keyed by a **published
   numeric function ID**. That makes every wrapper findable as an immediate push of its ID, with no
   symbols and no exports required.
2. **Count the direct callers of each wrapper separately.**
3. **Read the contrast, not the count.** A wrapper with zero callers proves nothing on its own. It
   becomes evidence when **sibling wrappers in the same binary do have callers** — that is what
   distinguishes "linked but dead" from "the whole family is dead stubs". In Alan Wake four of six
   stereo wrappers had callers and the decisive one had none, and the one with none had no absolute
   reference anywhere in the module and was not exported, so no other module could reach it either.
4. **State what the scan cannot see.** A caller count over `E8` rel32 calls and absolute immediates
   misses a call made through a runtime-computed pointer. That residual risk shrinks when the sibling
   wrappers establish a direct-call convention in the same binary, and a live breakpoint closes it.

**⚠️ Not every zero caller count is meaningful — check whether the API expects to be called at all.**
On the NVAPI stereo family specifically, `NvAPI_Stereo_Enable` is a **persistent, driver-wide user
setting**, not a per-session call — a well-behaved game leaves it alone entirely, so its absence is
not evidence of anything and should not be added to the "unused" tally that makes the decisive zero
meaningful. The general form: before reading a zero as a finding, confirm the function is the kind of
thing this class of well-written caller would be expected to call in the first place.

### ⚠️ The claim-hygiene lesson, which is the transferable half

The structural result — *seven genuine dispatch IDs, one with zero callers while four have callers* —
was verified on the machine. The **conclusion** rested on a second claim of a completely different
kind: that the ID with zero callers *is* the mode-setting function. **If the ID→name mapping is wrong,
the conclusion inverts.** The project caught this and re-tagged the two halves separately rather than
letting the verified half lend its confidence to the unverified one.

**That is the pattern to copy: when a static result depends on a lookup table you did not verify, tag
the structure and the naming at their true separate confidences.** Deriving a strong conclusion and
then discovering its foundation is `[reported]` is a much worse day than splitting them upfront.

**And check whether the table is public before assuming it is not.** In this case the local driver's
own id→name table was stripped, so the mapping could not be confirmed against the shipped DLLs — but
**NVIDIA publishes the complete mapping** in `nvapi_interface.h` in its public NVAPI repository. This
sweep read it and confirmed all six IDs the project relied on, so that mapping is now
`[reported 2026-09-01]` — confirmed against NVIDIA's own published header, a first-party source —
and the project's conclusion stands. The
general point: *"the strings are stripped from the binary"* is a statement about the binary, not about
the world.

---

## Both eyes from one recorded frame: resubmitting the game's own command buffers

`[measured 2026-09-01, n=1 game]` Generalised out of
[`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr) (Vulkan), and applicable to any explicit
API — Vulkan or D3D12 — where the application records command buffers and submits them.

The [stereo submission strategies](#stereo-submission-strategies) section frames the choice as native
vs sequential vs AFR. On an explicit API there is a fourth option that costs far less than it sounds:
**record nothing, and submit the game's frame twice.**

Submit the game's own recorded buffers with the left view in the uniform, let them complete, rewrite
the uniform, submit **the same buffers** again for the right eye. No command mirroring, no shader
work, and — unlike AFR — **no temporal mismatch**, because both eyes come from one recorded frame of
one simulation tick.

**Whether it is legal is a flag you can read.** A command buffer recorded with `ONE_TIME_SUBMIT` may
not be resubmitted. DOOM sets it on **none** of its eight per-frame command buffers and imports
neither `vkResetCommandBuffer` nor `vkResetCommandPool`, so its buffers are legally resubmittable.
Check this before designing anything else — it is a cheap read from a capture and it decides the whole
approach. Note the serialisation requirement: without `SIMULTANEOUS_USE` the first submission must
complete before the second begins, so this is sequential stereo, not parallel.

### 🚨 The trap that comes with it: a uniform buffer is usually a linear allocator

The approach needs the per-eye camera to be **writable between the two submits**, which means knowing
where it is. The obvious move — scan the mapped uniform memory once, record the addresses of every
camera copy, then patch those addresses each frame — **does not survive contact with a real renderer.**

DOOM's dynamic offsets climb **monotonically** through the frame and across frames, at roughly 137 KB
per frame: the uniform buffer is a **linear allocator**, so a given draw's camera slice sits at a
*different address every frame*. Measured consequence: a scan located 180 camera copies, and on the
following frames a verify-before-write guard passed **5 of 180, then 0 of 180**.

**Three things follow, and all three are general:**

- **Never cache a uniform-buffer address across frames.** Derive it each frame from the dynamic
  offsets the draws actually bind — which a hook on the bind call already sees.
- **Scan the window, not the buffer.** Only the ~137 KB written this frame can matter. Scanning the
  whole 64 MB allocation is both wasteful and, on write-combined memory, actively dangerous — see
  [never CPU-scan mapped GPU memory in place](#never-cpu-scan-mapped-gpu-memory-in-place--it-is-write-combined).
  In this case the full scan stalled the live game for roughly 2.7 seconds, twice.
- **⚠️ It retroactively invalidates earlier patching experiments.** An older result — "patching the
  camera across 72 blocks changed the image by only 1–2%, therefore the GPU-side camera is
  downstream" — was almost certainly **patching stale slots the GPU no longer read**. The conclusion
  drawn from it went back to `[hypothesis]`, and a later negative test at the submit path became
  **untested rather than disproved**, because with 5-then-0 successful writes it could not have
  produced a positive. This is the
  [silent no-op](#silent-no-ops-verification-that-cannot-see-the-failure) pattern with a moving
  target: *before believing a negative from a memory patch, prove the write landed where the GPU was
  reading **on that frame**.*

---

## D3D9 to a modern VR compositor: the shared-handle bridge, and its two traps

`[reported 2026-09-01, n=2 projects]` Researched by
[`far-cry-2-vr`](https://github.com/TefMeister/far-cry-2-vr) and applied by
[`enslaved-vr`](https://github.com/TefMeister/enslaved-vr). Not built or run by this account.

Every OpenXR/OpenVR compositor wants a D3D11+ (or Vulkan/GL) texture. A large share of the games worth
converting are D3D9. The bridge is a documented Windows interop path with **no CPU round-trip**:
create the texture on the **D3D11** side with the shared misc flag, take its `HANDLE` from
`IDXGIResource::GetSharedHandle`, and open that handle from **D3D9Ex** via
`IDirect3DDevice9Ex::CreateTexture(..., pSharedHandle)`. The game renders into the D3D9 side; the
compositor consumes the D3D11 side.

**The load-bearing word is *Ex*, and many D3D9 games never create such a device.** Three static checks
that fail in different ways, so their agreement means something — Enslaved failed all three:

1. the import table names `Direct3DCreate9` only, in the normal **and** the delay-import directory;
2. the string `Direct3DCreate9Ex` does not occur in the executable at all, which rules out a runtime
   `GetProcAddress`;
3. the `IDirect3D9Ex` / `IDirect3DDevice9Ex` / `IDirect3DSwapChain9Ex` IIDs occur **zero** times,
   which rules out a `QueryInterface` upgrade on a legacy-created device.

**The cheap route that remains — upgrade the device inside your own proxy.** If you already ship a
`d3d9.dll` proxy, it can call `Direct3DCreate9Ex` itself and hand the game the Ex object **through the
legacy interface**: `IDirect3D9Ex` derives from `IDirect3D9` and `IDirect3DDevice9Ex` from
`IDirect3DDevice9`, so a game compiled against the base vtable never needs to know. Interface
inheritance does the work; no wrapper objects, no vtable forwarding.

**🪤 Two traps, and both decide the design rather than tune it:**

- **`D3DPOOL_MANAGED` does not exist on a D3D9Ex device.** Any `CreateTexture` /
  `CreateVertexBuffer` / `CreateIndexBuffer` asking for it **fails**. So the silent upgrade above is
  only viable if the game's renderer never asks for MANAGED — and *that is not answerable
  statically*. It is the difference between a one-line proxy change and a resource-remapping project,
  so establish it first, with a logging proxy, before committing to the approach.
- **There is no D3D9 keyed mutex.** `D3D11_RESOURCE_MISC_SHARED_KEYEDMUTEX` has no
  `IDirect3D9KeyedMutex` counterpart, so the synchronisation primitive every D3D11 interop tutorial
  reaches for is unavailable to a D3D9Ex producer. The established substitute is an
  `IDirect3DQuery9` event query plus double or triple buffering.

**Relation to the alternative:** the
[whole-frame capture route](#capturing-the-finished-frame-the-whole-frame-route-to-a-headset) avoids
all of this by presenting a flat overlay, and pays for it with no stereo and no 6DoF. This bridge is
what you build when you have outgrown that ceiling.

---

## Never gate a state change on exact equality with a value that only *lerps* toward its target

`[reported 2026-09-01, n=1 — read from published upstream source]` Found by
[`visceral-re2-vr`](https://github.com/TefMeister/visceral-re2-vr) in a shipping VR framework's
first-person module, diagnosing a roughly one-second camera settle that this account's own releases
expose to users.

The bug earns a section because it is **invisible to code review and to a debugger's first glance**,
and because the fixed form is one line away from the broken one. Written as our own minimal
illustration, the shape is:

```text
target   = in_vr ? 0.0 : slider_value        // correct: VR wants exactly zero
smoothed = lerp(smoothed, target, k)         // k on the order of 0.0008 per frame at 60 fps
...
if (ready && smoothed == 0.0) { snap(); }    // a branch that will not fire
```

Everything reads correctly. The VR branch *sets the right target*. But an asymptotic approach in
floating point does not land on an exact `0.0` within any timescale a play session contains, so the
snap branch is gated on a condition the code's own update rule will never satisfy. Not "impossible in
principle" — but impossible in practice, which for a player is the same thing.

**The fix is not an epsilon.** It is the observation that the smoothed float was standing in for a
state the code already knows directly. In the case that produced this, the same file a few lines
earlier does the same job correctly by testing the VR state itself rather than inferring it from a
smoothed value. So:

> **Test the state; do not infer it from a signal that is merely converging on what the state
> implies.**

**⛔ And note the workaround that does not work**, because it is the first thing anyone tries: setting
the user-facing smoothing slider to zero changes nothing, since in VR the target is forced to zero
regardless of the slider and it is the *lerp*, not the slider, that is on the path. A setting that
appears related and provably is not is a good way to lose an evening — see
[prove the value you are debugging is the one the feature reads](#prove-the-value-you-are-debugging-is-the-one-the-feature-reads).

This is a specific instance of a guard already implied by
[a signal must be able to separate the states](#controls-a-negative-needs-a-positive-one-a-positive-needs-a-no-op-one):
**a threshold or equality test is only as good as the signal's ability to actually reach it.**

## Prove the test can fail: mutation-check a numerical verification before trusting it

`[verified-numerically 2026-09-02, n=1 project]` A stereo-maths change — per-eye viewports and per-eye
constants for a from-scratch render device — was verified **without launching the game** by compiling
the very header the DLL compiles into a standalone test, pushing its constants through the exact
arithmetic the GPU performs (vertex shader → perspective divide → viewport transform, in `float`), and
comparing the resulting pixel against ground truth built **independently**: the engine's own published
projection formula evaluated in `double` for a camera translated to each eye, plus the side-by-side
layout stated geometrically. Five frame shapes including a letterboxed one, several eye separations
including zero, both layouts, both eye-swap states, hundreds of random points each; plus invariants —
mono constants bitwise equal to the pre-change build, viewports two exact non-overlapping halves,
parallax **sign** and **magnitude**, zero separation giving identical eyes. Tens of thousands of checks,
none failing.

**That pass is a negative result — "no bug found" — and [rule 1 of the controls
section](#1-before-recording-a-negative-as-fact-confirm-the-test-could-have-gone-positive) applies to
it exactly as it does to a live probe: prove the test could have gone positive.** The way to do that is
to break the code on purpose and watch the test notice. Three mutations of a scratch copy of the header
were run through the unchanged test: eye-shift sign flipped, the cropped layout using the full frame
width, and the sub-rectangle origin dropped from the stereo offset. Each failed, and each failed in a
**different number of checks** — the third in exactly the letterboxed case, which is what told the
author that path was the one it exercised. A mutation run does two jobs: it proves the test has teeth,
and its failure *counts* localise which check guards which bug.

Three habits that made this cheap enough to do routinely:

- **Keep the maths in a header with no SDK or graphics-API types**, so a plain console test can compile
  the identical code the DLL ships. A test of a reimplementation tests the reimplementation.
- **Derive the ground truth in a different formulation and a different precision** from the code under
  test. The engine's own pixel formula in `double` and the shader pipeline in `float` cannot share a
  bug — the one outcome the test cannot catch is a derivation error common to both, and the project
  wrote that down as the single live outcome that would indict the maths.
- **Write the launch-outcome table before the launch.** With the arithmetic already proven, every
  possible wrong picture on the first run maps to a *code* bug — a constant not reaching the shader, a
  viewport not restored, a second constant buffer not bound — and the table says which. The maths is
  off the suspect list before anyone puts a headset on.

A design note worth carrying with it: **for a proof of stereo, do not model convergence.** Parallel eye
cameras with identical projections put zero disparity at infinity, which is what a headset compositor
expects (per-eye poses are translations; the asymmetric frusta come from the runtime's eye tangents
later). On a flat 3D display that puts everything "in front of the screen" — fine for a proof, not a
display tuning, and worth saying so nobody tunes it.

Generalised from [`unreal-gold-vr/modding-notes/`](https://github.com/TefMeister/unreal-gold-vr/tree/main/modding-notes)
(`2026-09-02-m2-stereo-proof-built-verified-deployed.md`); the test output and mutation record are in
that repo's `dev-archive/recon/`.

## When byte-identity is the evidence, the tree is read-only

`[verified-numerically 2026-09-02, n=1 binary]` A source tree rescued from a deleted repository was
proven to be the deployed build by rebuilding it locally and comparing the result **byte for byte**
against the DLL installed in the game — an exact comparison of the whole population, not a sample. That
single fact is the entire reason the rescued source is trustworthy, and it is destroyed by **any** edit
inside the tree, including well-meant housekeeping. Within hours a hygiene checker flagged a frozen
status document inside that tree for carrying an untagged claim, and the correct fix was to **change the
scanner to skip vendored and rescued trees**, not to add the tag. Work on such a source by **branching
from it**, never by editing it in place; if a tool ever complains about a file inside it, fix the tool.
Generalised from [`XIII2003-vr/engine-research/`](https://github.com/TefMeister/XIII2003-vr/tree/main/engine-research)
(dossier, 2026-09-02).

### ⚠️ Correction 2026-09-03: anchor the claim to a **commit**, not to the tree

The rule above is right about the evidence and wrong about how to protect it, and it failed within a
day of being written. The project it came from needed that tree — it was the only surviving source
for the work in hand — so within hours a development pass added files to it, edited four more, and a
later pass extended them. A fresh build of the tree no longer reproduces the installed DLL, and the
"read-only" instruction had simply been overtaken by the project's own needs.

**Nothing was lost, because git had already solved this.** The untouched tree is still exactly
reproducible at the rescue **commit**, and re-verifying the byte-identity claim means building *that
commit*, not `HEAD`. So the durable form of the rule is:

- **Anchor a byte-identity (or any reproduction) claim to a commit hash in the claim itself**, not to
  a directory. `[verified-numerically <date>, at <hash>]` survives everything a working tree does not.
- **Do not ask a live source tree to stay frozen.** A tree the project needs will be edited; a rule
  that forbids it will be broken by the people who most need the evidence to hold.
- The half that does survive: **do not tidy the frozen artefacts inside such a tree** to satisfy a
  hygiene checker. Teach the checker to skip vendored and rescued trees instead.

The general lesson is broader than rescued source: **a claim whose evidence is "the state of some
files" decays silently, and a claim whose evidence is a content hash does not.** Where a claim rests
on reproducing something, name the immutable thing it can be reproduced *from*.

## Per-draw stereo reaches only the draws that read the transform you hooked

`[inferred-static 2026-09-02, n=1 binary]` Generalised out of
[`XIII2003-vr`](https://github.com/TefMeister/XIII2003-vr) (Unreal Engine 2 / D3D8).

A tempting stereo design on a fixed-function-era API is **draw everything twice**: hook the draw call
and issue each batch once per eye, each with its own view and projection and its own half-width
viewport. It is attractive because it needs no camera hunt, and because — unlike patching a shared
transform setter — it has
[no early-out to defeat](#stereo-hazard-a-setter-that-early-outs-on-an-unchanged-matrix): the engine's
own cached matrix is never modified, so the cache behaves exactly as stock.

**The gap is that "its own view and projection" means different things for different draws.** A draw
issued through the **fixed-function** pipeline takes its transform from the API's transform state. A
draw issued under a **programmable vertex shader** takes it from **shader constants the game uploaded**,
and is completely unaffected by anything you do to fixed-function state. Those draws stay **mono**
while everything around them goes stereo — a silent, partial failure that presents as a depth oddity
on some objects rather than as a missing code path.

This is not hypothetical on old binaries. A whole-`.text` vtable-call scan of one UE2-era D3D8 render
device found the programmable path **present and used**: 10 `CreateVertexShader` sites, 17
`SetVertexShader`, 4 `SetVertexShaderConstant`, against 15 `SetTransform` — and one constant-upload
site pushes **five consecutive constants starting at register `c0`**, which is the shape of a 4×4
transform plus one spare. What is *not* statically knowable is the part that decides the design:
whether those draws carry **world geometry** or only skinning, terrain and effects, and what fraction
of a real frame they are.

**So the recon needs a denominator.** This is the practical rule, and it generalises past D3D8: an
instrument that counts traffic on the path you intend to hook cannot tell you what that path
**covers** — only what it sees. Counting `SetTransform` calls all day says nothing about the draws
that never call it. **Count the path you are not planning to hook in the same run**, and report the
two as a ratio: *fixed-function draws vs programmable-VS draws, per frame*. A zero on the second
number turns the cheap design into the right design; a non-zero one tells you what you would have
shipped broken. This is the same failure family as
[counting events instead of measuring content](#counting-events-is-not-measuring-content) — a number
that cannot express the answer you need.

**⚠️ And on D3D8 the obvious classifier does not work.** That API overloads a single `DWORD` for
**both** FVF codes and vertex-shader handles, so "this draw is programmable" cannot be read off the
value the game set. The heuristic that an FVF has its low bit clear is a runtime convention, not a
guarantee, and a design should not rest on it. Record instead what `CreateVertexShader` actually
**returned** and test membership of that set — and **ignore declaration-only creations** (created with
no shader function), which still run fixed-function and would otherwise inflate the programmable count
and condemn a design that was fine. Keep the bookkeeping off the hot path: bump one counter in the
draw hooks and do the set lookup in the far rarer shader-binding call.

**Report the instrument's own ceiling.** If the shader-handle table can overflow, say so in the log
and treat an overflowed run's programmable count as a **lower bound** rather than a measurement.

## Turn off the post-processes that re-derive the view before judging a stereo run

`[reported 2026-09-02]` Generalised out of [`enslaved-vr`](https://github.com/TefMeister/enslaved-vr)
(Unreal Engine 3 / D3D9), where a public 3D Vision fix for the same binary made the point explicitly.

A stereo injection that works at the **vertex** stage — patching a view or view-projection constant so
geometry is transformed per eye — does not touch the **pixel-stage** copies of that same matrix. Any
post-process that re-derives the view for itself therefore keeps running **monoscopically over a
stereo image**:

- **motion blur** (reprojects using view-projection, per pixel),
- **temporal AA**, **screen-space reflections**, **SSAO**, **depth of field**, and anything else
  reconstructing world position from depth.

The consequence for *testing* is the one that costs time: **a stereo run judged with these effects on
is judging an uncorrected pass, not your fix.** You will see smearing, ghosting or a doubled image
and attribute it to your separation value, your basis, or your injection point, when the geometry
underneath was correct all along.

**So the first line of any stereo test protocol is: turn them off in the game's own options or ini.**
It costs nothing, it needs no code, and it removes an entire class of misleading result before you
start tuning anything. Only once the geometry is confirmed correct is it worth deciding whether each
effect gets [fixed per eye or patched out](#temporal-effects-under-afr) for the shipped mod.

**A useful corollary about separation.** With the post-processing off, the project's own separation
figure stopped looking wrong: on an engine whose conventional world unit is 1–2 cm, a separation of 6
units is 6–12 cm — at or above a real interpupillary distance, i.e. **correct in magnitude**. The
"something is subtly off" symptom that had been read as a bad separation value was the uncorrected
pixel-stage pass. Rule out the passes you are not touching before you tune the number you are.

## The engine may have no projection matrix to patch

`[reported 2026-09-02]` Generalised out of
[`manhunt-2003-vr`](https://github.com/TefMeister/manhunt-2003-vr) (RenderWare).

Per-eye work is usually described as *"find the projection matrix and build an off-axis frustum"*.
Older and middleware engines frequently **do not store one** where you can reach it. RenderWare, for
example, expresses a camera's frustum as a **view window**: a pair of half-tangents,
`tan(fov/2)` per axis, held on the camera object beside an aspect ratio, from which the matrix is
built downstream at begin-update time. The camera's position and orientation live not in a view
matrix but in the camera's **frame** — a separate transform node.

**This is good news, not bad.** Where an engine keeps the frustum in this decomposed form, a per-eye
render becomes two edits in the engine's own vocabulary:

1. **shift the view window** to make the frustum off-axis for that eye, and
2. **translate the camera's frame** by half the interpupillary distance along its right vector,

both applied before the engine begins its update for that eye — no matrix to intercept, decompose,
or reconstruct, and nothing downstream to fight. It also means the search target changes: stop
hunting for a 4×4 near the camera and start hunting for **two floats and a frame pointer**, which are
much easier to identify by value and much harder to confuse with a derived output.

**Generalise the reflex, not the field names.** Before assuming an engine hides a projection matrix,
check what its camera actually stores: FOV plus aspect, half-tangents, a view window, near/far planes,
or a matrix. Whichever it is, that is the representation the renderer reads, and therefore the one to
edit — see [finding the camera matrix the engine actually
reads](#finding-the-camera-matrix-the-engine-actually-reads) for why editing a derived copy achieves
nothing.

## A public reimplementation of your game is a signature source, not just a reference

`[inferred-static 2026-09-02, n=1 game]` Generalised out of
[`psychonauts-vr`](https://github.com/TefMeister/psychonauts-vr), against **Astralathe** — a
GPLv3 mod loader and API extender for the same 2005 title. Nothing was copied, and it is not installed.

An open-source mod loader, decompilation or API extender for the game you are working on is
ordinarily treated as documentation. It is more useful than that: such projects have to locate engine
functions in the retail binary too, so they usually **publish byte signatures** — and a signature is
something you can run against **your own** executable.

**The method, and why it is stronger than reading their addresses.**

1. Take each published signature and scan your own binary for it. Record **how many matches** there
   are; a signature with one match is worth having, a signature with several is worth nothing until
   narrowed.
2. **A unique match at an address you had already identified independently is corroboration** — two
   parties, two methods, one address — and it also hands you the engine's **own name** for that
   function. That naming is often the real prize: it can settle whether the thing you hook is the
   engine's top-level per-frame entry point or an inner helper of it, a question that changes what
   your hook is guaranteed to see.
3. **A unique match at an address you had not found is a lead, not a fact.** It came from their
   binary analysis, not yours. Verify it in your own image before building on it — disassemble it and
   check it does what the name claims.
4. **Split the confidences.** Being able to compute an address is one claim; the *meaning* attached
   to what lives there is a different one, sourced from them. Tag them separately, exactly as with
   [an unverified ID→name table](#-the-claim-hygiene-lesson-which-is-the-transferable-half). In
   the worked case, a chain to the scripting VM's state pointer was confirmed structurally in our own
   binary while the claim about *which field of it* is the interpreter state stayed `[reported]` —
   and that is the half that would crash if wrong. Read it and print it before passing it to anything.
5. **Disassembling around a newly named function is where the free findings are.** Locating one
   visibility function this way immediately corroborated two unrelated prior claims from other
   sessions — a camera flags byte and an engine-wide culling toggle whose earlier measured null
   result the disassembly now *explained* (the flag gates only one branch, and an earlier check can
   return before reaching it). A confirmed null with a mechanism is worth far more than a confirmed
   null alone.

**⚠️ Two cautions.** Their addresses are for **their** build — match the version, and prefer
signatures over addresses for exactly this reason. And note anything self-recursive before you hook
it: a function that calls itself once per box face will make a naive counter report several times the
real number of objects.

**Licensing:** GPLv3 (or any licence) on their repository restricts **their code**. It does not make
a fact about your own binary unusable, and this method takes no code — it takes a pattern, runs it
against your own file, and keeps your own result. Read online, credit the project, copy nothing.

### One global turns `n=1` into `n=K` for free

A related trick from the same pass, and it costs one scan. If an address is a **singleton global** —
one engine object, one application object — then **every site in the binary that reaches a field off
it must encode that same base address**. So a claim resting on a single observed site can be upgraded
by scanning `.text` for the same operand pattern and counting. In the worked case a base-plus-offset
pattern matched **72 sites**, and re-running with the operand pinned to the candidate address matched
all 72 with none eliminated — the strongest corroboration any address in that project's dossier has,
obtained without launching the game.

Two conditions make it valid, and both must be stated with the result: the global really is a
singleton (two instances of the class would break the argument), and the match count is reported
**with** the pinned-operand re-run, since the unpinned pattern alone would also match any *other*
global used the same way.

## Match the engine's own accessor, not the ideal maths

`[inferred-static 2026-09-02, n=2 functions]` Generalised out of
[`psychonauts-vr`](https://github.com/TefMeister/psychonauts-vr).

When you read an object's world position out of a scene graph, the textbook answer is to compose the
parent chain. A worked case is a useful corrective. In that engine, the position **setter** converts a
world position into the node's local space through its parent, while the engine's own
`GetAbsPosition` and `GetPlayerPosition` accessors contain **no reference to the parent field at all**
— they return the node's own transform row directly. Getter and setter genuinely disagree.

**The conclusion is not "the engine has a bug to work around".** It is that **every script-facing
position read in that game already lives with this**, all of its own gameplay logic included — so a
mod that reads the position the same way is **exactly as correct as the engine is, and no more
wrong**. Composing the parent chain "properly" would make the mod disagree with the game about where
the player is, which is a worse failure than inheriting the game's own convention.

**The reflex to take:** when a transform read looks theoretically incomplete, go and read what the
engine's **own** accessor does before adding maths to it. If the engine doesn't compose, don't
compose — and record the one case that would break the assumption (here: an object that actually
acquires a non-null parent) as a **named, testable** condition rather than a vague caveat, so a single
live reading can retire it.

**⚠️ One scanning caution from the same investigation:** a whole-`.text` scan for writes to a struct
offset finds **unrelated classes that happen to share that offset**. Two of the hits there were a
texture pointer and an unrelated three-field write at the same displacement. An offset is not a type;
confirm the class before counting a site as evidence.

## Sources

- **XIII (2003) VR** (this account) — harness tick sites, the disproved render-path diagnosis, the log-before-the-call habit, and the exclusive-mode DirectInput wall that `SendInput` cannot cross; generalised out of [`XIII2003-vr/engine-research/`](https://github.com/TefMeister/XIII2003-vr/tree/main/engine-research) §9a/§9b; the byte-identity read-only-tree rule from the same dossier (2026-09-02)
- **Psychonauts VR** (this account) — the void-behind-the-player characterisation and measurement method, the camera-matrix identification arithmetic, the double-rotation trap, the unbound-key false negative and the camera-height-is-not-eye-height rule (notes 71–72); generalised out of [`psychonauts-vr/modding-notes/`](https://github.com/TefMeister/psychonauts-vr/tree/main/modding-notes) and [`psychonauts-vr/dev-archive/`](https://github.com/TefMeister/psychonauts-vr/tree/main/dev-archive)
- **Unreal Gold VR** (this account) — the mutation-checked numerical verification, the no-convergence and full-window-2D-layer design notes; generalised out of [`unreal-gold-vr/modding-notes/`](https://github.com/TefMeister/unreal-gold-vr/tree/main/modding-notes)
- **Visceral — RE2 VR** (this account) — the HMD-anchored body float and the pelvis-drop grounding fix; generalised out of [`visceral-re2-vr/modding-notes/`](https://github.com/TefMeister/visceral-re2-vr/tree/main/modding-notes)
- **RE Village sniper scope** (this account) — the argument-encoding silent-zero case, the hook-to-acquire-a-handle pattern, and the posted-window-message input route; generalised out of [`re-village-scope-vr/modding-notes/`](https://github.com/TefMeister/re-village-scope-vr/tree/main/modding-notes)
- **DOOM (2016) VR** (this account) — the launch-time gate and the date-match-your-evidence point, the line-endings false negative, the `strings` minimum-length trap, the in-process raw-input route, the call-argument-not-a-global switch shape, and the repeated-launch/ASLR sampling trap; generalised out of [`doom-2016-vr/external-research/`](https://github.com/TefMeister/doom-2016-vr/tree/main/external-research) and [`doom-2016-vr/modding-notes/`](https://github.com/TefMeister/doom-2016-vr/tree/main/modding-notes)
- **Alice: Madness Returns VR**, **Alan Wake VR**, **Prince of Persia (2008) VR** and **Burnout
  Paradise VR** (this account) — the third-party-stereo-fix-as-intelligence method, the proxy-export
  completeness rule and its static-vs-dynamic failure modes, and the instrument-can-be-the-bug case;
  generalised out of each project's `engine-research/` folder:
  [`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr) ·
  [`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr) ·
  [`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr) ·
  [`burnout-paradise-vr`](https://github.com/TefMeister/burnout-paradise-vr)
- **Arcade Controls for RE2 VR** (this account) — the signal-cannot-separate-the-states guard;
  generalised out of [`arcade-controls-re2-vr`](https://github.com/TefMeister/arcade-controls-re2-vr)
- **Enslaved VR** (this account) — the post-processing-before-judging-stereo rule and the shipped
  comfort-switch habit; generalised out of [`enslaved-vr/engine-research/`](https://github.com/TefMeister/enslaved-vr/tree/main/engine-research)
- **Manhunt VR** (this account) — the projection-matrix-free frustum (view window plus camera frame);
  generalised out of [`manhunt-2003-vr/engine-research/`](https://github.com/TefMeister/manhunt-2003-vr/tree/main/engine-research)
- **LukeRoss00** — additionally his 2020 [SteamVR discussion-board
  report](https://steamcommunity.com/app/250820/discussions/8/3001046778344834329/) on per-view poses
  being mishandled by that runtime, and the workaround published with it
- **SirKandela** (Chaos LTD) and **Rylie Pavlik** — the 2023 [Khronos forum
  thread](https://community.khronos.org/t/oculus-runtime-ignores-projection-layer-views-pose/110078)
  reporting the opposite runtime behaving the opposite way
- **eqzitara** and the HelixMod community — the published 3D Vision fix for Enslaved (2013), read
  online for its pass list and its motion-blur requirement; no code taken
- **Jill (`scrunguscrungus`)** — **Astralathe** (GPLv3), whose published function signatures are the
  worked example of scanning someone else's signatures against your own binary: <https://gitlab.com/scrunguscrungus/astralathe>
- **Fire-Head** — **MHWSF**, the Manhunt widescreen fix whose published camera globals named the
  view-window representation; read online, verified independently, no code taken
- **HelixMod community** (incl. **Chiz**) and the **geo-11 / 3D Vision fix scene** — their published
  per-game fix write-ups, changelogs and settings documentation, read online as reports on engine
  behaviour. No code taken. <https://helixmod.blogspot.com/>
- **Remleo** — [UEVR PR #433](https://github.com/praydog/UEVR/pull/433), the optional-truthiness/garbage-vtable-slot gamma fix (merged 2026-08-30)
- **ErwinGunsmith** — [REFramework PR #1809](https://github.com/praydog/REFramework/pull/1809), restoring the `false` return of `on_pre_gui_draw_element` (merged 2026-08-28)
- **prideslayer** and contributors — **VRIK Player Avatar** (Skyrim VR), cited only to distinguish the familiar VR floor-calibration/height-offset problem from the pose-dependent float described above: [nexusmods.com/skyrimspecialedition/mods/23416](https://www.nexusmods.com/skyrimspecialedition/mods/23416)
- **UEVR** render modes (native / synchronized-sequential / AFR) — [docs.uevr.io](https://docs.uevr.io/) · [github.com/praydog/UEVR](https://github.com/praydog/UEVR)
- **Luke Ross R.E.A.L.** — AER (alternating eye rendering); *technique reference only* — the GTA V repo is unlicensed (view-only, don't reuse code) and other titles are paid: [patreon.com/realvr](https://www.patreon.com/realvr) · [github.com/LukeRoss00/gta5-real-mod](https://github.com/LukeRoss00/gta5-real-mod)
- **starfield2vr** (mutars) — Reflex-marker timing, keep-and-fix-TAA per eye: [github.com/mutars/starfield2vr](https://github.com/mutars/starfield2vr)
- **anvilengine2vr** (mutars) — two-hook timing, disable-TAA, basis round-trip: [github.com/mutars/anvilengine2vr](https://github.com/mutars/anvilengine2vr)
- **vrframework** (Elliott Tate) — the framework these techniques are described against: [github.com/elliotttate/vrframework](https://github.com/elliotttate/vrframework)
- **NVIDIA** — published developer documentation for 3D Vision Automatic (the clip-space shader
  footer, the per-game profile requirement, the post-processing/deferred caveat) and the NVAPI stereo
  headers that define Automatic vs Direct mode:
  [3D Vision Automatic background](https://archive.docs.nvidia.com/gameworks/content/technologies/desktop/nv3dva_background.htm) ·
  [stereoscopic issues](https://archive.docs.nvidia.com/gameworks/content/technologies/desktop/nv3dva_stereoscopic_issues.htm) ·
  [nvapi_lite_stereo.h](https://github.com/NVIDIA/nvapi/blob/main/nvapi_lite_stereo.h)
- Inspection tools: [RenderDoc](https://renderdoc.org/) · [PIX](https://devblogs.microsoft.com/pix/)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
