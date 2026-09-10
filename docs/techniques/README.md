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
- [A D3D9 `Reset` can disarm a device hook, silently and late](#a-d3d9-reset-can-disarm-a-device-hook-silently-and-late)
- [Test a runtime-compiled shader without the game: one file, two compilers](#test-a-runtime-compiled-shader-without-the-game-one-file-two-compilers)

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


**Two more cautions, added 2026-09-03 from two UE3/Remedy-era 3D Vision titles:**

4. **The driver feature a dormant path talks to may itself be dead.** NVIDIA 3D Vision — the target
   of every "Stereo3D" menu toggle of the 2009–2013 PC generation — was discontinued in April 2019;
   **425.31 is the last driver that includes it**, DX11 stereo survived to 452.06 via workarounds and
   was removed with the RTX 30-series launch driver, and only DX9 is *reported* to still work on
   current drivers `[reported 2026-09-03]`. So on a current machine the game's own stereo checkbox
   will typically do nothing — not because the path is stripped, but because the other end is gone.
   Details and sources in [generic drivers](../generic-drivers/README.md#-3d-vision-itself-is-discontinued--what-that-means-for-a-games-native-stereo-toggle).
   This is *good* news for the estate's approach: both titles examined turned out to be pure
   **consumers** of driver-published separation and convergence (Automatic mode, no `SetDriverMode`,
   no `SetActiveEye`, getters only), so a proxy that supplies those values itself drives a stereo
   path the shipping shaders already implement, with no NVIDIA driver in the loop.
5. **Before adding a per-eye edit at a second stage, check whether the shipped shaders already apply
   it there.** Alice's pixel shaders carry NVIDIA's `x' = x + S·(w − C)` compiled into retail —
   28,017 of 43,025 branch on the stereo-enabled constant and 14,479 read the parameters texture —
   so shearing the pixel-stage matrix *as well* would apply the offset twice. `[inferred-static
   2026-09-03]` A dormant stereo path may be partly live in the bytecode even when it is dead at the
   driver, and the stages are usually asymmetric by design.

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

`[reported 2026-09-02]` — read directly from Khronos's own published `openxr.h`, first-party but a document read rather than a measurement — following up a
`[hypothesis]` two sibling projects independently reasoned their way to hours apart on the same day.

**The problem this answers.** A same-frame stereo submission (needed by AER and by any true two-eye
render) wants to hand the runtime two independent poses, one per eye, in the same frame. On
**OpenVR**, [issue #1253](https://github.com/ValveSoftware/openvr/issues/1253) — filed by
**LukeRoss00**, the author of the AER technique this library already documents, describing exactly
this wall — is **still open**: `IVRCompositor::Submit` is called once per eye, and SteamVR keeps only
the pose from the **last** call, discarding the first. A same-frame two-eye submission over OpenVR
therefore cannot carry two different poses at all.

**Re-read 2026-09-04** `[reported 2026-09-04]`: opened 2019-11-23, **last activity 2020-04-22** (a
community request for a team update), eight comments, **not one from anyone at Valve**. Two details
worth carrying that a one-line "still open" hides:

- **A partial fix was reported and never documented.** The filer reported in the thread that the
  underlying bug had been fixed in a SteamVR beta **for the lighthouse driver only**, not for the
  Oculus or WMR backends, and not mentioned in any public changelog. So "OpenVR cannot do this" is
  right as a portable design constraint and **may be wrong on one specific driver** — which is worse
  than a clean no, because it means a test on one headset can pass and mislead.
- **No workaround for the general case exists in the thread.** Nobody has posted a way to submit two
  distinct per-eye poses that works across runtimes.

The design consequence is unchanged and firm: **treat one shared pose per frame as a constraint of the
OpenVR submission path**, not as a shortcut you chose, and do not infer from a passing test on one
driver that the constraint has lifted.

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

## Alternate-eye rendering: latch the eye WITH the frame, or you silently swap them

`[verified-numerically 2026-09-04, n=1 project, 22 assertions]` Alternating eyes across frames — the
AER pattern — has one hazard that is worth knowing before you write it, because it produces a picture
that looks fine.

**The shape of the bug.** The eye is chosen in one place and the finished image is captured in
another, and in a single `Present` hook those two places are separated by the flip to the next eye.
Frame *N* is drawn with eye `E_N`; the hook flips the state to `E_(N+1)`; only then does the
submission path capture frame *N*. **Reading "the current eye" at capture time therefore reads the
wrong one, every frame** — the images are correct and the labels are off by one, so the left eye is
handed the right image and vice versa.

**⚠️ What makes it dangerous is that swapped eyes are not obviously broken.** Both eyes get a real,
correctly-rendered view; the disparity is the right magnitude and the wrong sign. It reads as **working
stereo with inverted depth** — a comfort complaint, a "the scale feels odd" note, something you might
attribute to IPD or world scale and spend a session tuning. Nothing errors, nothing is missing, and no
frame counter is wrong.

**The fix is a latch, not a re-read.** When the frame completes, record the eye it was **drawn** with
before flipping, and have the submission path read that latched value rather than the live state.
Treat "no per-eye offset was applied to this frame" as an explicit third value meaning mono, so an
un-offset frame is not silently attributed to an eye.

**And test it against the picture, not against the intent** — the useful assertion is that the latched
eye matches **the sign of the offset actually written into the matrix** for that frame, which catches a
sign convention flipped anywhere in the chain rather than just confirming your own bookkeeping agrees
with itself.

Generalised from [`far-cry-2-vr`](https://github.com/TefMeister/far-cry-2-vr), 2026-09-04, where the
per-eye submission path is built and self-tested and **has not been run**.

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


### Name the wrapper first — on Steam it is usually Steam's own, and then unpacking is a static step

`[measured 2026-09-03, n=3 projects]` The section above assumed the fix is always to read live
memory. That is the universal fallback, but three projects in this estate have now hit an encrypted
`.text` and two of them found something cheaper: **the wrapper was Steam's own DRM stub (SteamStub),
applied at upload time, and public open-source unpackers restore `.text` on disk without running the
game.** That keeps the whole job in the no-game-running tier — on one project it moved a blocked scan
from "needs a launch" back to "static, do it now" purely by naming the packer.

**The identification, in three cheap reads, none of which runs anything:**

1. **Section names.** A section called **`.bind`** is SteamStub. (`.vmp0`/`.vmp1` is VMProtect;
   `.themida`/`.winlice` is Themida.) The wrapper section is usually the last one in the file.
2. **Entry point outside `.text`** — something else runs first, and `.text` is very likely encrypted
   until it does.
3. **The two statistics already described above** — `.text` entropy at the 8.00 ceiling and zero
   `CC` padding runs.

Then the version tell: **SteamStub v3.x carries a header magic `0xC0DEC0DF` inside `.bind`**, and
the stub's own code validates it (a `cmp dword [reg+4], 0xC0DEC0DF`), so finding that constant is
conclusive. **Its absence is not** — the v2.x stub has no such marker. Alice: Madness Returns
(2011) was v3.1 with the magic; Prince of Persia (2008) was **v2.1 with no magic** and all the other
tells present, and Steamless unpacked both (`.text` entropy 8.00 → 6.7 / 6.6, `CC` runs reappear,
`.bind` gone, entry point back inside `.text`). ⚠️ **A `steam_api.dll` import is a positive-only
witness**: one of the two wrapped games has **no Steamworks API at all**, only the wrapper — its
absence proves nothing.

**Three consequences worth more than the unpacking itself:**

- **A packed `.text` retroactively voids every code-search negative ever recorded against that exe.**
  Strings in `.rdata` were always readable, so string-based findings stand; but any *"no such
  instruction / immediate / call in `.text`"* result was a search of ciphertext and could not have
  returned a positive. One project had leaned on exactly such a negative for a design decision.
  Audit the dossier for them the moment a wrapper is identified.
- **Expect the unpacked file not to launch.** It generally will not run standalone, because the game
  still expects the environment the stub set up. That is the expected outcome, not a failed unpack —
  the goal is a readable section, not a second way to start the game. Work on a **copy**, never the
  shipped exe, and do not commit the result: it is game content.
- **Re-run the positive control on the unpacked copy before believing anything from it.** On Alice
  the still-packed copy was scanned side by side as a negative control and returned zero for every
  id *including the control* — the false negative demonstrated rather than argued. A partial unpack
  would fail the same way.

And once the code is readable, do not expect small-integer immediates to be informative in either
direction — the unpacked Prince of Persia image carried 1,196 and 592 occurrences of the two
enum values a session had been hunting, because those are ordinary integers. A name-driven registry
leaves no literal by construction; the search belongs in the data files, not the exe.

Sources: [atom0s/Steamless](https://github.com/atom0s/Steamless) and
[GHFear/Steamstub-v3-Unpacker](https://github.com/GHFear/Steamstub-v3-Unpacker) (both explicitly
for software you own); Adam Hlt's
["Cube World Reversing — Unpack the game"](https://adamhlt.com/cube-world-reversing-unpack-the-game/)
for the `.bind` / entry-point / encrypted-at-rest description. Generalised from
[`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr),
[`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr) and the Manhunt
case study, 2026-09-03; the identification test was first written up by a `/gr` pass on Alice.

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

### An affordance reachable only from a GUI panel is invisible to a driven session

`[measured 2026-09-05]` Generalised out of
[`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr).

A capability that exists only as a button in a framework's overlay panel **does not exist** for a session
with nobody at the mouse — and nothing errors to say so. One project carried a texture-cycling probe for a
week, built and working, and could not use it in any automated run because it had only ever been wired to a
panel button. It answered a question that was blocking the project the whole time.

Two consequences worth building in from the start:

- **Every capability needs a headless entry point** — a console command, a config-file key, a hotkey the
  input layer can synthesise. The panel button is a convenience on top of that, never the only door.
- **Assume the overlay cannot be clicked from outside the process.** In that project the framework's
  overlay registered hover but ignored synthetic clicks, via both `mouse_event` and
  `SendInput` with `MOUSEEVENTF_ABSOLUTE` `[disproved 2026-09-05, n=3 attempts]`. If the panel is the only
  door, the room is locked.

**A related cost worth pricing in:** where a script cannot be reloaded safely — that project's reload
duplicates every frame-callback registration — a one-line script change costs a **full relaunch**. That
turns "try it and see" into a budgeted action and is a strong argument for
[making one launch answer many questions](#make-one-launch-answer-many-questions).

**And one launch trap from the same automation work:** issuing a Steam URL launch (`steam://rungameid/…`)
while the Steam client is still starting produces **no process and no error at all** — the request is
silently dropped. Wait for the client before launching, and treat "no process appeared" as a launcher
result rather than a game result.

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

### When a scan finds nothing, check its BASE before you widen its RANGE

`[verified-live 2026-09-04, n=1 project]` A worked failure, because the wrong fix was both real and
convincing.

A scanner that learns the camera's location by matching a known value found **zero** hits. Inspection
turned up a genuine defect: its scan window was **6.6× smaller** than the span it needed to cover, so
it had been reading a fraction of the region. That was fixed, the window widened, the build deployed —
and the next run **still found zero**, having scanned the full span with no truncation at all.

The actual cause was the **base**, not the range. The scanner asked the process for its *largest*
memory mapping and scanned that. A by-value search then located the camera copies — sixty-four of
them, a packed position and a clean view matrix — clustered near the start of a **different** region
entirely. The scan had been reading the right offsets in the wrong buffer, and no amount of widening
could have corrected that.

**Two rules come out of it, and the second is the one that costs sessions:**

- **A scan has three parameters, and range is the least likely to be wrong.** Base, stride and range:
  check that the base is the region actually holding the data (locate it once by value), and that the
  stride matches the layout, before touching the range. "Largest mapping" is a heuristic for *where a
  game keeps big things*, not for *where this value lives* — a process can hold several equally large
  mappings.
- **⚠️ A real defect found while diagnosing a symptom is not thereby the cause.** The undersized window
  was a true bug and fixing it was correct; it simply was not the reason for the zero. The tell is
  that the fix was verified by *compilation*, and belief in it ran ahead of the run that tested it.
  Until the failing path has actually been exercised again, a fix explains nothing — the same trap as
  [an early-out that stops the failing path being exercised](#stereo-hazard-a-setter-that-early-outs-on-an-unchanged-matrix),
  and the reason this account tags a claim by how it was established rather than by how convinced its
  author was.

The instrument was worth keeping either way: the widened window now prints the scanned span against
the full one and warns when the ceiling is still exceeded, so the next zero comes with its own
diagnosis. Generalised from [`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr), 2026-09-04.

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

### Reflection gets you to "N unnamed slots"; disassembly names them by use

`[inferred-static 2026-09-03, n=651 shaders, 1 title]` The paragraph above ends with "watch that
buffer for a slot that changes when the camera moves but stays constant across draws within a
frame". One project built exactly that probe, ran it once, and then — with the game closed —
disassembled the same shaders it had only reflected, and found that three things the probe design
had taken for granted were wrong. All three transfer to any D3D10/11 title whose bytecode ships with
reflection intact, so they are recorded here rather than in that project's dossier alone.

- **The same cbuffer name at two sizes is more likely two STAGES than two passes.** A reflection
  census showed the shared buffer declared at 512 bytes by 186 shaders and 2,352 bytes by 465, and
  the smaller one had been read as "the shadow-pass variant" because it carried three cascade
  matrices. The `RDEF` chunk also records the *program type*, and sorting on it split the census
  cleanly: every 512-byte declaration is a vertex shader and every 2,352-byte one a pixel shader.
  One C-side buffer, two stage-specific views of it. Check the stage field before naming a layout
  after a pass.
- **A bound buffer can be larger than the shader's declaration, so a live size that no shader
  declares is not a mystery buffer.** `[reported]` Microsoft's own reference for
  [`VSSetConstantBuffers`](https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11devicecontext-vssetconstantbuffers)
  says a bound resource may exceed what a shader can address and the shader simply sees its declared
  prefix. So when the live probe saw a 3,136-byte buffer created in lockstep with the 512-byte one
  and no shipped shader declaring 3,136 bytes, the parsimonious reading is the pixel-side allocation
  being bigger than the 2,352 bytes its shaders declare — `[hypothesis]` in that project until a
  bind census confirms it, but the *shape* is ordinary D3D11 behaviour, not evidence of a hidden
  layout.
- **A run of four camera-varying slots is not a matrix until an instruction multiplies by it.** The
  probe had flagged slots 16..19 as the shared view-projection because they changed with the camera
  and sat in a 4×4-shaped block. Disassembly showed slots 16 and 17 read as an `xyz` offset plus a
  `w` scale for a projected coordinate in thirteen vertex shaders, and slots 18 and 19 read by
  **nothing**. The clip transform was at slots 0..3: a `mul` / `mad` / `mad` / `add` chain on the
  input position ending in `SV_Position`, in fifteen shaders. **Shape is layout coincidence; use is
  meaning.** Walk the register chain back from the position output and list which cbuffer slots feed
  it — that is the only list that names "the camera" with any authority.

The transferable habit is cheap: **disassemble before you design the probe, not after you read its
log.** Reflection is a table of contents; the instruction stream is the text. It works through
Denuvo-class protection for the same reason reflection does — the bundle is data, not code — and on
651 shaders it ran in about a minute. The tool that did it, `dxbc-usage.py` in
[`flat-to-vr-RE-toolkit`](https://github.com/TefMeister/flat-to-vr-RE-toolkit) (splits a bundle by
stage, disassembles with Microsoft's `fxc -dumpbin`, tallies `cb<N>[slot]` reads per register, and for
vertex shaders walks the chain from `o0` back to the cbuffer slots), is ours and free to use.

**⚠️ Correction 2026-09-04 — that walk must respect PROGRAM ORDER, and ours did not.** The
back-walk from the position output is only as good as its notion of what actually reached the output.
Shader registers are **reused**: `r0` can carry a term into `SV_Position` early in the program and
something entirely unrelated later. A walk that collects every cbuffer slot ever seen in a register on
the position chain, without regard to whether the write happened *before* the position was written,
**over-reports** — ours did, by thirty-four rows on one census `[inferred-static 2026-09-04]`.

The concrete damage, and the reason this is worth a paragraph rather than a changelog line: the
over-report listed a run of slots as feeding the position that turned out to be a **falloff/blend
block and a projector space written to a texcoord**, not a transform at all. That is precisely the
"a 4×4-shaped run is shape, not meaning" trap this section exists to warn about, arriving one level
up — **in the tool built to avoid it.**

**Two things follow.** First, when a census lists a slot, check what that slot's result is actually
*written to* before believing it; a slot whose value ends up in `o3` is a texcoord, whatever its shape.
Second, the [re-audit rule](#the-instrument-can-be-the-bug) applies to your own analysis tools as much
as to your hooks: when the tool was fixed, its unaffected sections were re-run and reproduced
**byte-for-byte**, which is what allows the earlier conclusions drawn from them to stand rather than
all needing to be re-derived. Record that check; without it a tool fix invalidates everything it ever
produced.

Generalised from [`mad-max-vr/modding-notes/`](https://github.com/TefMeister/mad-max-vr/tree/main/modding-notes)
(`2026-09-03c-the-two-layouts-are-vertex-and-pixel-and-the-camera-matrix-is-per-pass.md`).

### ⚠️ A "constant across every draw in the frame" filter excludes a per-PASS camera by design

`[verified-live 2026-09-03b, n=1 title]` for the write cadence; `[inferred-static 2026-09-03c]` for
what the rows are. The by-value probe described two sections up carried a filter that seemed
obviously right: the view matrix is one thing per frame, so keep only slots that are identical across
every draw in the frame and vary between frames. On the same title the engine writes its shared
buffer **about ten times per frame — once per pass** — and the camera rows differ per pass (shadow
cascades, reflection, main). The filter therefore excluded the actual clip transform **by
construction**, and could only ever have found per-frame quantities such as the camera *position*,
which is exactly what it reported.

Two rules follow, and neither costs a launch:

- **Decide the filter from the fill cadence, not from the concept.** If the buffer is written more
  than once per frame, "constant within the frame" is the wrong invariant; the right one is
  "constant within a *write*, and the write whose view origin matches the main camera is the one you
  want".
- **Log per write, not per frame.** A per-frame histogram cannot show a per-pass matrix at all; the
  per-write dump is where it appears, so it is not an optional verbosity level. The follow-up probe
  on that project flags the write whose per-pass origin slot equals the frame-constant camera
  position — one dump answers both "which write is the main pass" and "what is in it".

This is the third member of a family this page already holds: the
[early-out hazard](#stereo-hazard-a-setter-that-early-outs-on-an-unchanged-matrix) (a setter that
skips unchanged uploads), [counting events instead of content](#counting-events-is-not-measuring-content),
and now a filter whose invariant the engine does not hold. Each is an instrument whose design
encodes an assumption about cadence, and each fails silently when the engine's cadence differs.

Generalised from the same `mad-max-vr` note as the section above.

### When a game compiles its shaders decides how you read its constant map

The recurring question — *which register or constant carries the view-projection, and what is it
called?* — is answered by a different technique depending on **when the game turns shader source
into bytecode**, not on which engine or graphics API it uses:

| When the game compiles | What ships on disk | How to read the constant map | Seen on |
| --- | --- | --- | --- |
| Ahead of time, source shipped | HLSL/`.usf` **source** | just read it — reserved-register comments are usually right there | Enslaved (UE3 ships `Common.usf`; `c0` = `ViewProjectionMatrix`) |
| Ahead of time, source stripped | compiled bytecode **with a reflection block** | parse it — D3D9 bytecode carries `CTAB` naming every constant/register, D3D10+ carries `RDEF` | Alice: Madness Returns (45,832 `CTAB` tables), Enslaved (34,046), Mad Max (`RDEF`) |
| At runtime | HLSL plus a shipped **shader-compiler redistributable** | **hook the compiler** — proxy `d3dcompiler_4x.dll` and log every source string, entry point and define as it compiles | **none in this estate** — the case first filed here (Alan Wake) was `[disproved 2026-09-03]`: it ships pre-compiled CTAB after all; see the correction below |

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


**⚠️ Corrected 2026-09-03 — the third row's worked case was wrong, and the inference behind it is
the lesson.** Alan Wake's retail install turned out to ship **62 pre-compiled shader containers
(~16 MB) with `CTAB` intact, in a plain folder outside its archives** — 9,971 constant tables — so
it belongs in the **second** row, not the third, and the off-disk method that carried Alice and
Enslaved carried it too `[disproved 2026-09-03]`. The chain *"ships a compiler ⇒ compiles at runtime
⇒ no cache to read ⇒ proxy the compiler"* broke at every arrow, and three static checks separate the
cases before anyone builds a proxy:

1. **Does the redist folder say anything?** A `DirectX\` folder full of `D3DCompiler_4x` cabs is
   usually the **stock DirectX redistributable**, shipped verbatim by nearly every DX9-era title —
   Alan Wake's is the complete June-2010 redist, 154 cabs spanning 2005 onward. Count them; a whole
   historical redist describes Microsoft's installer, not the renderer.
2. **Which entry point does the game actually call?** `D3DXCompileShader*` (D3DX9) and `D3DCompile`
   (`d3dcompiler`) are different seams, and D3DX9 delegating to D3DCompiler internally does **not**
   make the latter the game's call site. Proxying the wrong one intercepts nothing.
3. **Do shader SOURCES ship?** The decisive check, and it costs a directory listing. No `.hlsl`,
   `.fx`, `.usf` or engine-specific source — and especially a `...FromFile` entry point with nothing
   to point at — means the compile path is a **developer/fallback affordance with no inputs in a
   retail install**. The error strings are real code; that does not make the path reachable.

The general form: **presence of a capability in a shipped binary is evidence about the build
system, not about runtime behaviour.** Retail builds carry dev-only paths — editors, hot-reload,
recompilation — no player reaches. Before concluding "it must do X at runtime", look for the *inputs*
X would need; missing inputs is far stronger evidence than a present code path. (The mirror error is
covered elsewhere on this page: an *absent* capability is not proof either when `.text` is packed or
the call is resolved by string.) The third row therefore currently has **no worked case in this
estate**; it stays as a real possibility, not an observed one.

#### A fourth case: the shader is ASSEMBLY TEXT, and it ships in the binary

The table above has three rows because those were the three cases seen. A D3D8-era title added a
fourth `[inferred-static 2026-09-04, n=1 binary]`: its renderer DLL contains **no shader bytecode at
all** — no version tokens anywhere in the file — but does contain three `vs.1.0` **assembly source
strings**, which the game hands to the runtime assembler at device-init time. Six shader handles came
from those three sources.

That is the friendliest case of the four, and it is easy to miss precisely because a bytecode scan
comes back empty and reads as "this game ships no shaders":

- **The register semantics are written out.** Reading the text gave the constant map directly — the
  transform occupying `c0..c3` in row-vector order, a colour in `c4`, two lookup vectors in `c5`/`c6`
  — with no disassembler, no reflection block and nothing running.
- **It tells you what each shader is *for*.** One source pushes the position along the normal by a
  scalar and emits a flat colour: that is an outline pass, identified by reading it rather than by
  elimination from a frame capture.
- **⚠️ And it carries the usual trap: not every shipped string is reachable.** The same DLL holds six
  **Xbox-format** pixel-shader strings the PC assembler cannot accept — console-port leftovers. Same
  rule as everywhere else on this page: presence in the binary is evidence about the build, not about
  what runs. Match the strings to the call sites that reference them before believing any of them.

**How to check for this case:** search the binary for the assembly-language mnemonics themselves
(`vs.1.0`, `vs.1.1`, `ps.1.`, `dcl_`, `mov r`, `dp4`) as plain ASCII, not for bytecode signatures. It
costs one grep, and on a fixed-function-era title it is worth doing **before** planning any runtime
capture.

#### If both pipelines read the same transform, per-eye stereo is one edit

The more valuable half of that same static read was not the shader text but where its constants came
from `[inferred-static 2026-09-04]`. Both upload sites composed the register block by multiplying
**the three transforms the fixed-function path was already being given** — world, view and projection,
each cached by the renderer's own `SetTransform` handler — and uploading the product.

That collapses a problem this library usually treats as two. A mixed-era engine draws some geometry
through the fixed-function pipeline and some through shaders, and the reflex is to plan two separate
per-eye interventions. But if the programmable path is uploading `W · V · P` built from the same
cached `V` and `P`, then **replacing `V` and `P` with their per-eye versions covers both pipelines**:
the fixed-function draws through the API's own transform setter, the programmable draws by
recomposing the same product. No matrix inversion, no shader patching, nothing to defeat.

**So the question to ask of any such engine is narrow and static:** *where does the constant block
come from?* Follow the upload call backwards to its arguments. If it is composed from cached
transforms, you have one lever; if it arrives from somewhere else entirely, you have two problems and
should know that before writing either.

**And verify the identity at the point of use, rather than assuming it.** What is established
statically is that the code *composes* the product; whether the value in the register at draw time
still equals your shadowed `W · V · P` is a separate question — another pass, a later upload, or a
path you have not read can break it. The pattern that project shipped is worth copying: at draw time
compare the uploaded block against the shadow, rewrite per-eye **only on a match**, and on a mismatch
**draw the geometry unmodified and increment a counter**. Never drop the draw. That way a wrong
assumption costs a mono object and a number in the log, rather than a hole in the world and a
mystery — the [silent no-op](#silent-no-ops-verification-that-cannot-see-the-failure) inverted into a
loud one.

Generalised from [`XIII2003-vr`](https://github.com/TefMeister/XIII2003-vr) (`engine-research/`,
2026-09-04), whose per-eye maths was itself
[mutation-checked](#prove-the-test-can-fail-mutation-check-a-numerical-verification-before-trusting-it)
before any launch.

#### The register is not fixed: a skinning palette displaces the camera constants

`[inferred-static 2026-09-03, n=2 engines]` Two unrelated D3D9 engines, read the same day, put the
same matrix in **two different registers depending on whether the shader is skinned**: Alan Wake's
projection sits at `c0` in 2,238 shaders and `c192` in 2,084, because a 192-register skinning
palette occupies `c0..c191` in every skinned shader (n=1,954, zero counter-examples); Prince of
Persia (2008)'s fused `g_WorldViewProj` sits at `c0` in 6,292 and `c128` in 2,016, because a
128-register `g_Bones` palette occupies `c0..c127` in exactly those 2,016. Any proxy that assumes one
register corrupts the other half of the corpus — silently, since the write lands on a valid bone
matrix. The robust pattern on both is the same: **parse the reflection block out of the bytecode at
shader-creation time and keep a per-shader register map at runtime** — the same data the off-disk
scan reads, read again where it is authoritative. The converse does not hold (unskinned shaders can
also sit at the displaced register), so the register does not identify a skinned shader.

Generalised from [`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr) and
[`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr), 2026-09-03.

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
- **⭐ And the dictionary reaches past the type table.** `[measured 2026-09-07]` On that same engine,
  **UI screen names are stored as CRC32 little-endian in the shipped menu-definition files** — not
  just types in a registry, but ordinary content references. Once you have a CRC32 dictionary built
  from the executable's own identifier strings, try it against **every** unexplained 32-bit constant
  you meet: immediates in vtable methods, fields in data files, keys in lookup tables. The same
  session also recorded a clean **negative** from the same dictionary — a hypothesised
  "state-hash channel" between two subsystems does not exist, and that negative is only worth
  anything because the dictionary was demonstrated to resolve other constants in the same sweep.
- **⚠️ But an engine usually hashes names at ONE level of its hierarchy, not everywhere.** Finding a
  name hash somewhere is not evidence that the object you want is hash-addressed. On a different
  engine, four independent public sources agree that animation *motions* are addressed by a plain
  numeric `(bank, motion)` pair, while the murmur-hashed fields in the very same file format are for
  **bone** names, and the one hash-keyed lookup in the API is at **file** level
  `[reported 2026-09-07, 4 sources]`. A session that assumed "this format hashes names, so my motion
  is keyed by its name hash" would have built the wrong lookup and read the failure as a format
  problem. **Establish which level is hashed before designing against it** — and note that the two
  readings usually predict *different* failure signatures, which is what makes the question cheap to
  settle.

Generalised from `prince-of-persia-2008-vr`'s static `.forge`/Scimitar decoding sessions of
2026-09-02 and 2026-09-07 (no launch), and — for the hashed-at-one-level rule — from
`visceral-re2-vr`'s 2026-09-07 motion-format research; engine-specific layout detail stays in each
project's own dossier.

### ⭐ A reflection table often carries the developers' own doc comments — search THOSE, not the names

`[verified-numerically 2026-09-05, n=57,214 field records]` The best single recon result of the
2026-09-05 sweep, and it is one grep away in any engine that ships reflection metadata.

The question was whether a 2016 engine's view structure carries a per-eye field — the thing every
stereo conversion needs to find or replace. Enumerating the whole reflection database gave a clean,
near-exhaustive **negative on names**: of 57,214 field records, **zero** contain `stereo`, and all 59
occurrences of `eye` are gameplay, AI or animation fields. Every candidate view class was enumerated
in full and none of them has a per-eye member. On names alone, that is where the trail stops.

**But the database also stores the engine programmers' own comments**, and searching *those* for the
same word returned exactly six hits, five of them renderer, and they answered the question outright:
the eye selector is an **index on a per-view object** (*"determines which viewColor image will be
rendered to, and which view from world will be used"*), the frame info holds a **fixed-capacity list
of two** world views (*"two identical ones in stereo-3D"*), and the single surviving per-view stereo
scalar is a **GUI** offset (*"for stereo 3D, the guis can be offset differently in each screenView"*)
with no world-camera counterpart. That last one is the structural reason a per-eye camera field was
never going to be found — the design puts the eye in *which view you are rendering*, not in a field
on the view.

**Why it generalises.** Reflection metadata exists to drive editors, serialisation and script
binding, and the comment strings are there because a tooltip in the editor needed them. They are
written by the people who built the thing, in the vocabulary of the feature rather than the
vocabulary of the code, and **a feature that has been renamed, half-removed, or folded into another
concept usually still says so in a comment** long after the identifier stopped mentioning it. Any
engine with a reflection or property system is a candidate: id Tech's field tables, Unreal's
`UProperty` metadata, a managed type database, an editor schema shipped alongside the game.

**Two disciplines to keep it honest, both learned here:**

- **Scope the negative to the population you actually enumerated.** These tables list *reflected*
  members only — one class showed 35 reflected fields inside 5,616 bytes — so the result is exact
  for *"is there a reflected eye field"* and merely strong for *"is there an eye field"*. Say which
  one you mean; they are different claims and only the first was measured. Same rule as
  [not mixing populations](#-and-do-not-mix-the-populations-when-you-quote-a-percentage).
- **Run a positive control on the enumerator.** The same scan re-found, unprompted, every field the
  project already knew by other means — the view origin and axis, the field-of-view pair, the
  explicit-projection-matrix override and its boolean — which is what makes the zero-hit result a
  measurement rather than a silence. And widening the validator changed the record count by **+0**,
  which bounds how much the parse could have been missing.

**The second prize is a struct signature you can verify a memory hit with.** Once the layout is
known, a value scan that finds one field also knows what must sit around it — a plausible
field-of-view pair at one negative offset, 0-or-1 booleans at two others, an orthonormal axis at
another. A hit can then be **confirmed** as the struct you wanted instead of assumed, which is the
difference between a candidate address and an anchor. It also makes any *other* field of that struct
directly addressable for the first time, including ones a project has been calling its highest-value
live test for weeks.

**One caution that came with it.** These particular tables turned out to have **no code references
at all** — neither absolute nor RIP-relative, checked at five addresses with a scanner that can see
both (see [the ModRM hole](#-a-cross-reference-scanner-that-does-not-decode-modrm-is-blind-on-x64--and-every-no-xrefs-result-it-produced-is-suspect)).
So a reflection database can be a complete map of the data and give you **no static entry into the
code that uses it**. Those are separate wins; getting the first does not imply the second.

Generalised from [`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr), 2026-09-05 (static
only, nothing launched); the engine-specific offsets stay on the
[id Tech 6 page](../engines/id-tech-6.md) and in that project's dossier.

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


**Measured 2026-09-03, same project, one session — the early-out is RARE, and the requirement is
unchanged.** `[measured 2026-09-03]` Cumulative would-fire counts: world **0.27 %** (3,536 of
1,325,094 sets), view **0.98 %** (3,536 of 361,272), projection **0 %** — the engine never re-sets an
identical projection. That bounds how often vanilla play *reaches* the early-out; it does not retire
the guard, because the hazard was never frequency. It is that a cache left holding *our* modified
matrix makes the engine's next legitimate set look redundant. What the number changes is the
**cost** of honouring the requirement — small — so the transform-setter route is no longer the
expensive option it looked.

## Enumerate every CPU write path to a constant buffer before believing your coverage

`[measured 2026-09-04, n=167 shaders]` A per-draw constant patch is only as complete as the set of
**write paths** it shadows, and on D3D11 there is more than one. A project whose patcher covered the
large shared world buffer found a large minority of its geometry uncovered, and the reason was neither
the shaders nor the patch mechanism — it was **which API call the buffer is filled through**.

| Buffer usage | How the CPU writes it | What a patcher must hook |
| --- | --- | --- |
| `DEFAULT` | `UpdateSubresource` — the only CPU write path such a buffer has | that call |
| `DYNAMIC` | `Map(WRITE_DISCARD)` then `Unmap` | both, copying the mapped contents at `Unmap` while the pointer is still valid |

**The instructive part is that the filter which hid the second population was correct.** The pool
registered only `DEFAULT` buffers and rejected `DYNAMIC` ones explicitly, on good grounds — its shadow
was fed by `UpdateSubresource`, which a dynamic buffer never sees. The rule was right for what it did
and wrong as a definition of coverage, and nothing in the logs said so, because a buffer nobody watches
generates no events at all. **A registration filter is a claim about the world; check it against the
world.** On that project, of the matrix-bearing shaders in the live table, **42% declared their
constant buffer at one of the sizes the coverage test had shown carrying real geometry** — all of them
already had complete reflected layouts recorded, so they had been patchable all along, and only the
buffer was out of reach.

**The fix is another shadow source, not another patch mechanism.** Hook the second write path, copy
into a shadow, and let the existing draw-time path run unchanged. Two details from the implementation
worth stealing:

- **Partition the existing arrays rather than duplicating them** — a disjoint index range for the new
  population means every invariant already reviewed for the first applies to the second unmodified, and
  the hot lookup does not slow down.
- **⚠️ Mind pointer width in your lock-free bookkeeping.** That project caught a pending-map table
  claiming its slot with a **32-bit** interlocked compare-exchange on a **64-bit** resource pointer,
  which truncates. It was found before it shipped; a truncating claim would have matched the wrong
  buffer occasionally and produced corruption no log would explain.

Generalised from [`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr), 2026-09-04,
where the second path is built, guarded by compile-time assertions **proved to fire** by deliberately
breaking their values, and **not yet run**.

### ⚠️ And do not mix the populations when you quote a percentage

The same project states, correctly and in the same breath, that the new path addresses **42% of the
shader table** and that a different figure describes the **draws** in a frame. Those are different
populations, and neither converts into the other: a handful of shaders can draw most of a scene, and a
large fraction of the shader table can be responsible for very little of it.

**Whenever you write a coverage percentage, name what it is a percentage of** — shaders, draws,
vertices, or frame time — because the reader's decision usually depends on the *draw* or *time*
figure while the number that is easiest to compute is the *shader* one. This is the
[counting events is not measuring content](#counting-events-is-not-measuring-content) rule applied to
your own reporting rather than to the engine's.

### 🚨 And an in-flight map's identity is `(context, resource)` — never the resource alone

`[verified-numerically 2026-09-05]` The first attempt at the `Map`/`Unmap` path above failed
completely on its first live run — **zero** shadow writes against **2,787,733** table overflows — and
the reason is a rule worth stating flatly, because it is the natural design and it is wrong.

To patch a buffer written through `Map`/`Unmap`, you must remember, between the two calls, which
pointer belongs to which buffer. The obvious table is keyed on the **resource pointer**: claim a slot
at `Map`, find it again at `Unmap` by scanning for that pointer. That works exactly as long as one
buffer can only be mapped once at a time — and in a deferred-context renderer it cannot.

**`WRITE_DISCARD` renames per context.** Several deferred contexts may legally hold an outstanding
map of the *same* `ID3D11Buffer` simultaneously; that is the whole point of the mechanism. With
around six deferred workers plus the immediate context and a few dozen registered buffers, the same
buffer holds several in-flight maps at once — so a scan keyed on the buffer matches somebody else's
entry, the table saturates, and it never drains again. The failure is total rather than partial,
which is at least an honest symptom.

**The key that is unique by construction is the pair.** The API permits one context only one
outstanding map of a given subresource, so `(context, resource)` identifies an in-flight map
exactly. A fixed open-addressed table hashed on both halves, linear-probed with a bounded probe
budget, replaced the linear scan.

Two details that matter if you write one, both about a lock-free table being read while it is
written:

- **Claim in one order and release in the other.** Claim by compare-and-swapping the resource field
  first and publishing the context field **last**; free by scrubbing context and data first and
  releasing the resource field **last**. A prober that matches on *both* fields can then never
  match a half-written entry, in either direction.
- **Check the pointer width of what you compare-and-swap.** A 32-bit interlocked compare-exchange on
  a 64-bit resource pointer compiles, runs, and aliases two different buffers whenever their low
  words coincide. Caught here before it shipped, which was luck rather than process.

**The verification method is the transferable half, and it needed no game.** The test harness
`#include`s the *shipped* hash and table header rather than a transcription of it, and — the part
worth copying — it **replays the old design and reproduces the original failure**: 32 slots placed,
346 of 378 maps overflowed, one buffer holding seven entries at once. Only then does it run the new
one: all 378 placed, worst probe depth six, one buffer hashing from seven contexts into seven
distinct buckets, every `Unmap` paired to its own pointer, and the table draining to empty over
20,000 rounds. **A fix whose harness cannot reproduce the bug has not been shown to fix anything** —
it has been shown to pass a test, which is a different and much weaker statement.

**What was explicitly *not* claimed**, and is the right way to leave it: the shadow copy behind the
table still holds one window per buffer, so two contexts writing the same buffer at once still
cannot both be represented. That limitation is unchanged, and its counter only starts meaning
anything now that writes reach it at all. There is also a live possibility the counters cannot yet
distinguish — `Map`/`Unmap` were hooked **once, on the immediate context, and never late-hooked**,
while a neighbouring entry point was; deferred contexts come from a different vtable, so "fixed" and
"our hook never sees these calls" look identical until a counter separates them. That counter was
added in the same change, which is the correct order.

**⚠️ And `unmaps == 0` has a SECOND cause, which is semantic rather than a hooking gap.**
`[reported 2026-09-07]` The natural reading of "we counted maps but no unmaps" is *our `Unmap` hook is
blind to these buffers* — a different vtable flavour, or a hook installed once on the immediate context
and never late-hooked. That is a real failure mode. But on a **deferred** context `Map` does not touch
the real resource at all: the runtime hands back a **fresh scratch allocation owned by that context's
command list**, committed when the list is replayed. In at least one public implementation of that
contract, **`Unmap` on a deferred context is a no-op and the update is committed in `Map`** — discard
allocates a new slice, and no-overwrite requires a prior discard on that list or errors.

So `unmaps == 0` can mean *there is nothing for `Unmap` to do*, and attributing it to a blind hook
sends the next session after an instrumentation bug that does not exist. **The discriminator is cheap:
count whether `Unmap` is reached on *any* context at all, not just on the buffers you care about.** A
global count that is also zero means the hook; a global count that is healthy means the semantics.

**And this is the reason the `(context, resource)` key above is right, stated positively rather than
as a bug post-mortem:** the same constant buffer can legitimately be mapped **simultaneously on two
different deferred contexts**, so a resource-only key is not merely lossy, it is incorrect by the API's
own contract. The cleanest public statement of why a per-*thread* structure is on the wrong axis comes
from the vendor's own documentation — *"only one thread can call a `ID3D11DeviceContext` at a time"*.
**Threads are unbounded and transient; contexts are few and stable.** Key on the thing the API
serialises.


Generalised from [`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr), 2026-09-05.
See also [deferred-context renderers](#deferred-context-renderers-finding-the-world-and-patching-it-once-per-eye).

## Dating a dependency: a fix newer than your build is not evidence that you are affected

`[reported 2026-09-04]` Modding frameworks move faster than the builds people run on them, so "a bug
was fixed upstream last week" arrives regularly. It is worth knowing what that does and does not tell
you, because the wrong reading in either direction costs something: chasing a bug you never had, or
trusting a build that has one.

**The question a commit message cannot answer** is whether a fix repairs a **long-standing** defect or
a **recent regression**. If long-standing, a build older than the fix has the bug. If a regression
introduced while adapting to something new, an older build never had it and upgrading would be a step
sideways. The commit title reads identically in both cases, and short of reading the source — which
this account does not do for other people's projects — it stays open.

**What you can settle instead is the blast radius, and it is usually cheap.** Rather than resolving
the unresolvable, ask which of *your* code touches the surface the fix concerns. On the worked case,
a run of upstream fixes to array handling and to string-versus-number ambiguity looked alarming until
someone checked: **none of the five scripts in the shipped release reads a managed array at all**, and
the only iteration in the shipped set walks a plain local table. The exposure was in the **recon
probes**, not the product `[inferred-static 2026-09-04]` — a materially different conclusion, reached
without resolving the original question at all.

**And note which half of your work the exposure lands on.** In that case it lands on
reconnaissance — whose entire product is an answer — and a data-model bug there **returns a wrong
value rather than an error**, which is the worst available shape and the reason the
[silent no-op](#silent-no-ops-verification-that-cannot-see-the-failure) rules apply to research code
as much as to a shipped hook.

**The three practical habits, in the order they pay:**

1. **Record the exact revision beside every finding** that depends on the framework, the way a game
   patch version is recorded. On these families "it worked yesterday" is a statement about two
   programs, not one, and the estate's confidence tags already have room for the clause.
2. **Date-check before doubting your own code.** When something behaves oddly, compare your build's
   date against the window of the known upstream regression before rewriting anything.
3. **Know that "release or master" is a false choice.** A third state is common and easy to forget:
   a **fork build**, taken for a feature neither upstream branch offers. Its version is a commit date
   and nothing else — and a fork that publishes no releases will not show up in any release check you
   run.

Generalised from a modding-lane verdict on
[`visceral-re2-vr`](https://github.com/TefMeister/visceral-re2-vr), 2026-09-04, answering a question
this library had raised twice.

### 🚨 The version that moves is usually the GAME's — and a moved struct field crashes with your framework nowhere in the stack

`[reported 2026-09-05]` — read directly from the merged pull request and the repository API, not from
a summary. The section above is about *your build* of a framework going stale. This is the more
common and much nastier direction: **the framework is fine, your build is fine, and the game shipped
a patch that moved a field the framework reads by hard-coded offset.**

A worked public example, merged upstream on 2026-09-05:
[REFramework PR #1822](https://github.com/praydog/REFramework/pull/1822) by **porlock2**. A March
2026 update to one RE Engine title re-laid-out its `via.render.Texture` object to match a sibling
title's layout — the description field moved to a different base, and the graphics-API resource
container moved by 0x18 bytes. The framework kept reading the old offsets. Four lines of change fixed
it (a per-title branch selecting the new values), and the contributor states the offsets were
**measured rather than estimated**, which is the right standard for a number of that kind.

**The symptom is the transferable part, because it actively points away from the cause.** The crash
happened at the publisher logo during startup, on **game worker threads, with no framework frame
anywhere in the call stack**, whether or not the framework's optional features were enabled. Nothing
about it says "a mod is reading a struct at the wrong offset". Everything about it says "this game is
broken" or "this driver is broken".

**Four things to take from it:**

- **A framework's offset table is a per-title, per-game-version assumption**, not a property of the
  engine. Two titles on the same engine can disagree, and one title can disagree with itself across a
  patch. Treat "engine X support" as "engine X, these titles, these versions".
- **When a game updates and something starts crashing, check the framework's offset accessors before
  you debug your own code.** That is a one-file read and it is upstream of every other hypothesis.
- **A crash with none of your frames in the stack is not evidence that none of your code is
  involved** — a bad pointer computed from a wrong offset is dereferenced by whoever uses it next,
  which is usually the engine, on its own thread. This is the crash-time sibling of the
  [co-occurring log line](#a-log-line-that-co-occurs-with-a-failure-is-not-an-explanation-of-it):
  where the fault surfaces says little about where it originates.
- **Know which side of a game patch your framework build sits on.** A fork build whose date precedes
  a title's layout change carries the pre-change path by definition, and no release check will say
  so. Combined with the fork rule above, the version question is properly three-sided: your build,
  upstream, **and the game**.

Credit: **porlock2** (the fix and its measurements) and **praydog** (REFramework). Surfaced by the
2026-09-05 `/sr` web sweep; no code was copied and nothing was installed to check it.

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

## Determine the matrix storage class two ways before writing any per-eye edit

`[verified-numerically 2026-09-03, n=3 projects, two classes]` The clip-space eye offset
`x' = x + S·(w − C)` is the workhorse of D3D9-era stereo and appears in several projects here. In
matrix language it is *"row 0 gets `S` times row 3 added, then `S·C` subtracted from its last
entry"*. That is correct **as mathematics** and dangerously incomplete **as instructions**, because
"row 0" is not a place in memory. Where it lives depends on the shader's **matrix storage class**,
and two projects in this estate landed on opposite sides of it on the same day:

| | Alan Wake (Remedy) · Prince of Persia 2008 (Scimitar) | Alice: Madness Returns (UE3) |
| --- | --- | --- |
| `CTAB` class | `D3DXPC_MATRIX_ROWS` | `D3DXPC_MATRIX_COLUMNS` |
| register *i* holds | row *i* | column *i* |
| bytecode shape | four `dp4 out.n, c[i], v` — full dot products | `mul r, c1, v.y` then three `mad`s accumulating a whole float4 |
| `clip.x` is | `dot(c0, v)` | the `.x` **lane** across all four registers |
| the per-eye edit | `c0 += S·c3` ; `c0.w −= S·C` | `c[i].x += S·c[i].w` for all *i* ; `c3.x −= S·C` |

**The two implementations are transposes of each other.** Porting one onto the other mixes columns,
corrupts `clip.w`, and **still renders** — the worst failure mode, because the result is a plausible
wrong picture rather than an error. Alice's test suite transplants the Alan Wake implementation
verbatim and shows it diverging (`ndc.x` off by ~0.03) with `clip.w` corrupted; mutation testing on
both suites (twelve and six mutants, all caught, controls pass) shows they genuinely discriminate.

**The check is cheap and needs no game running — do both halves:**

1. **The reflection block states the class outright.** D3D9 `CTAB` records a `TypeInfo` per constant
   whose first `u16` is the `D3DXPARAMETER_CLASS` (`2` = rows, `3` = columns). Most tooling ignores
   this field; for stereo work it is the single most load-bearing thing in the table.
2. **The bytecode is the confirming second read.** Four consecutive `dp4`s against consecutive
   registers ⇒ registers are rows (column-vector `M·v`). A `mul`/`mad` chain accumulating one float4
   from `v.x`, `v.y`, `v.z`, `v.w` ⇒ registers are columns (row-vector `mul(v, M)`).

The metadata alone can be stale or mis-set by a toolchain, and the bytecode alone can be ambiguous
in a shader that uses only part of the matrix. Alan Wake's `g_mLocalToView` is the worked case for
needing both: declared `4×4`, occupying **3** registers — consistent only with a `[0,0,0,1]` fourth
row elided and translation living in each row's `.w`, which the class field cannot tell you and the
`dp4` + `mov r.w, v.w` pattern settles at once.

This generalises past stereo: **any** code that patches a shader-constant matrix in flight — FOV
override, camera detachment, projection hacks — has to answer the same question first, and the
failure is always a plausible-looking wrong picture. Pair it with
[the register-displacement trap](#the-register-is-not-fixed-a-skinning-palette-displaces-the-camera-constants):
together they are the two ways a "just write the matrix" plan silently breaks.

### On a fused matrix, `p00` cannot be recovered under object scale — keep the camera's projection

`[verified-numerically 2026-09-03, n=2 projects]` The clip-space form needs the projection's own
`p00` (the `[0][0]` focal term) to scale the separation. When the engine hands over a **standalone
projection** it is right there — and Alan Wake shows the elegant consequence: express the convergence
shear through the projection's *own* `row0.x`, and the game's FOV, near and far never have to be
recovered, so the edit stays correct when the game changes FOV at runtime. When the engine hands over
only a **fused** `World→Clip` (Prince of Persia 2008 has no standalone projection at all; Alan Wake's
particle, foliage and terrain paths bypass its split), the tempting recovery `p00 ≈ |row0.xyz|` is
exact on a rigid `P·V` and **demonstrably wrong the moment the object carries scale**, because the
scale is baked into the same row. Both projects' suites assert both halves. The rule: **supply `p00`
from a cached camera projection you captured elsewhere, never recover it from a fused matrix**; and
note that a fused draw's *attribution* — camera pass or shadow/light pass, the same shader serving
both — is a runtime question that reflection cannot answer, so a fused edit needs a pass
discriminator (the active render target) before it is safe.

Generalised from [`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr),
[`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr) and
[`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr), all 2026-09-03;
the storage-class table was first filed by the modding lane as an inbox drop.

### ⭐ And then the edit itself is ONE ELEMENT, not a rebuilt matrix

`[verified-numerically 2026-09-04, n=33 Python cases + 26 C assertions, 1 project]` The section above
says where the focal term must come from. This one says how little you have to do with it, and it is
the most economical result this library holds on per-eye rendering.

Take the row-vector convention (`clip = pos · M`, the one you establish with the
[storage-class check](#determine-the-matrix-storage-class-two-ways-before-writing-any-per-eye-edit)).
A per-eye camera is a translation of `d` along the **view** x axis, so `V_eye = V · T` and

```
M_eye = W · V · T · P  =  M + W · (V·T − V) · P
```

`(V·T − V)` has exactly one non-zero entry, `[3][0] = d`. For any **affine** `W` — fourth column
`[0,0,0,1]ᵀ`, true of every object transform — the product preserves that shape, and post-multiplying
by `P` turns it into *row 3 gains `d` × row 0 of `P`*. Row 0 of a perspective projection is
`[w, 0, 0, 0]` for a symmetric **and** for an off-centre frustum alike, because an off-centre frustum
carries its shift in row 2. So the whole per-eye edit is:

```
M[3][0] += d * w        // w = the horizontal focal term, from the SHARED matrix
```

**Why that is better than rebuilding `V_eye · P`, which is the obvious plan:**

- **It reads and writes nothing depth-related.** No row 2, no near/far, no `Q`. Every reversed-Z,
  infinite-far or unusual-clip-convention question simply cannot affect it — and those are exactly
  the questions that stay `[hypothesis]` longest on an unfamiliar engine. One project dropped a plan
  that depended on its reversed-Z reading being right, and on stepping around an unexplained
  per-position clip-z constant, in favour of this.
- **It is one float instead of sixteen**, so there is no transcription surface and no
  decompose-recompose round trip to lose precision in.
- **It works unchanged on the per-object path.** Where the matrix is `W · V · P` rather than `V · P`,
  it is the same element, the same `d` and the same `w` — no second derivation when that path is
  built.

**⚠️ The one trap, and it is the section above restated:** `w` must come from the **shared** matrix.
`|column 0|` of a *per-object* matrix has the object's scale baked in — a 3×-scaled object reads
3.54 where the true `w` is 1.18 — so taking `w` from the matrix you are editing silently scales that
object's stereo separation by its own size. Cache `w` from the camera's own transform, and assert it.

**Two things this does not give you.** The convention matters: under column-vector storage the same
argument lands on the transposed element, so establish the storage class first rather than trying
both. And the algebra being proven is not the picture being right — the project above has the
derivation, the write path and 63 self-test assertions, and **has not yet rendered a frame with it**.

#### ⚠️ It is a DIFFERENT stereo, not a shorter way to write the same one — and one condition on preferring it

`[verified-numerically 2026-09-04, n=2 projects]` A second project re-derived the above for its own
column-major layout, confirmed it reproduces exactly, and then found the more important thing: **the
one-element edit was already sitting in its shipped code as the constant term of a two-line shear.**
Comparing the two forms across six vertices from 12 to 8,000 units of depth, they differ in NDC x by
**exactly a constant**, and not at all in `y` or in `clip.w`. So:

```
NVIDIA-style shear  =  the one-element edit  +  a constant NDC shift
                    =  a parallel eye translation  +  convergence re-centring
                    =  off-axis (asymmetric-frustum) stereo
```

**Say that in those words, because "the edit is one element" reads as a simplification of the same
stereo when it is actually a simpler, DIFFERENT stereo** — the on-axis or parallel case, without
convergence. On a flat screen that difference is the entire comfort story. For an HMD it usually does
not matter, because the runtime's own per-eye frusta supply the asymmetry — but know which situation
you are in before choosing, rather than discovering it from the picture.

**⚠️ And prefer the one-element form only where nothing downstream already implements the
two-parameter form.** If the engine ships a pixel or post stage that applies
`x + separation · (w − convergence)` itself, reading both parameters out of a stereo-parameter
texture, **your vertex stage is not free to choose its formula**: it must match, or the two disagree
by a constant. That failure mode is a nasty one — **the geometry moves and every screen-space effect
stays put.** On the project that raised this, **28,017 shipped pixel shaders implement that form and
no vertex shader does** `[inferred-static 2026-09-04]`, an asymmetry that is deliberate in
3D Vision-era titles and cannot be edited away, because the bytecode is not ours to change.

**⭐ This is the same expression this library already documents as
[the clip-space stereo footer](#the-clip-space-stereo-footer-geometry-stereo-without-ever-finding-the-camera)** —
NVIDIA's own convention, which its documentation describes the driver appending to every vertex shader
it saw. What the Alice result adds is the half that page does not lead with: **a game can implement the
same formula in its own shipped shaders**, and where it does, the driver's absence does not remove it.
The companion NVIDIA page describes how those shaders are handed the two values — a small
**stereo-parameter texture** whose first texel carries the final separation in one channel and the
convergence in another
([stereoscopic issues](https://archive.docs.nvidia.com/gameworks/content/technologies/desktop/nv3dva_stereoscopic_issues.htm),
re-read 2026-09-04; both pages were already cited by this library for the footer itself). `[reported]`
from a vendor source; nothing was copied from either page.

**So the check to run first is cheap, static, and now has a name to search for:** do the shipped
shaders sample a **stereo-parameter texture**, or read a separation/convergence pair from a constant? A
game whose pixel shaders do this was built against 3D Vision Automatic and is correcting its own
screen-space work — and since that bytecode is not yours to change, your vertex stage has to speak the
same language. Where nothing downstream corrects, which is the case on the project the technique came
from, the one-element form is exactly right and every advantage above stands.

Verdict returned by the modding lane on
[`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr), 2026-09-04,
against its own 54-configuration harness.

Generalised from [`mad-max-vr`](https://github.com/TefMeister/mad-max-vr) (`engine-research/` §7a,
2026-09-04), where it is proven against independently multiplied `W`, `V` and `P` for ordinary,
reversed-Z-infinite and off-centre projections.

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

### ⭐ The inverse: a legal-but-unnecessary call is not evidence of a mechanism

The section above is about a call that looks successful and did nothing. This is its mirror, it
misleads in the opposite direction, and it is the more flattering of the two because it arrives as a
large number:

> **A call that is permitted but unnecessary still appears in a trace, in volume, and proves nothing
> about how the data actually got there.**

A counter is evidence that a call *happens*. It is not evidence that the call is the **mechanism** —
and the bigger the count, the more convincing the wrong conclusion feels. Instrumentation naturally
produces the first reading and readers naturally hear the second.

**The worked case** `[reported 2026-09-04, primary source]`. A Vulkan session counted **27,462**
`vkFlushMappedMemoryRanges` calls against the memory region holding the camera copies, and flagged it
as contradicting an earlier finding that the buffer is `HOST_COHERENT` and therefore not updated
through the flush path. There is no contradiction. The Vulkan specification says of
`VK_MEMORY_PROPERTY_HOST_COHERENT_BIT` that *"the host cache management commands
`vkFlushMappedMemoryRanges` and `vkInvalidateMappedMemoryRanges` are not needed to manage availability
and visibility on the host"*
([Memory Allocation chapter](https://docs.vulkan.org/spec/latest/chapters/memory.html), read directly
2026-09-04) — and **"not needed" is not "not allowed"**. Nothing forbids the call. An engine that
flushes unconditionally, without branching on memory type, produces exactly that count while the flush
does no work, because the write was already visible before it was made. **The count was compatible
with both hypotheses it was being read as evidence for**, so it could not discriminate between them.

**The trap is not Vulkan's.** Any API with an **optional or advisory** call has the same shape: D3D
`Flush`, redundant state-setting, predication, state blocks re-applying values already set,
`vkInvalidateMappedMemoryRanges` on coherent memory, and every "hint" entry point the runtime is free
to ignore.

**The discriminator is the same every time and it is usually one field.** Find the property that
decides whether the call *could* have mattered, and read it — here, the memory type's property flags:
coherent means the flush was ceremonial, non-coherent means the count is real evidence. That read is
free, and it decides which of two redesigns to build.

Generalised from a `/gr` research hand-off on
[`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr), 2026-09-04; specification text is the
Khronos Group's, read online, nothing copied.

### A read-back that returns the same number under every write is three hypotheses, not one

`[verified-live 2026-09-05, n=2 materials, n=3 launches]` Generalised out of
[`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr).

You write `0.0` to a shader or material parameter and read back `0.1`. The reflex is to call it
**re-assertion** — the engine is writing the value every frame and you are losing the race — and to open
a hunt for the writer. That hunt can consume a session, and it rests on a reading the observation does not
support. At least three explanations fit:

1. The engine **re-asserts** the value each frame.
2. The value is **clamped** on the way in, and `0.1` is the floor.
3. **The write never lands at all** — wrong instance, wrong encoding, or an API that reports success and
   discards.

A single value cannot separate them, and one value is all a single write ever gives you.

**The ladder that separates them costs one launch.** Write three values spanning the suspect bound and log
each read-back:

| written → read | reading |
| --- | --- |
| `0.500 → 0.500`, `0.050 → 0.100` | **a clamp.** No writer exists; cancel the hunt. |
| `0.500 → 0.100` | **re-assertion or a dead write** — still two; see below. |
| `0.500 → 0.500`, `0.050 → 0.050` | the value is yours, and something *later in the frame* overwrites it. Only now is finding the writer the right next step. |

That project's ladder returned `0.500 → 0.100` on both materials, which killed the clamp reading — a clamp
at 0.1 would have passed 0.5 through — and left two.

**⭐ The discriminator between the last two is a hold, not a write.** Set the value **every frame for a
second or two** and read it back. A per-frame writer loses to a per-frame hold; if the read-back is still
unchanged, re-assertion is dead and what remains is that the write never lands. That is what happened, on
two materials across two launches, and it closed the row: *there is no writer to hunt*. The parameter is
read-only through that API, driven from somewhere the material does not expose, or the instance being
written is not the one the renderer samples — and all three of those are addressed by a different route,
not by more searching.

**The transferable part is the shape.** When every observation of a quantity is the *same* observation,
you have one data point and several hypotheses. Design the smallest set of writes whose read-backs differ
between them, and only then spend a session on the hypothesis that survives. See also
[prove the value you are debugging is the one the feature reads](#prove-the-value-you-are-debugging-is-the-one-the-feature-reads).

## A D3D9 `Reset` can disarm a device hook, silently and late

`[verified-live 2026-09-03, n=2 resets, 1 title]` A D3D9 proxy that had been feeding a per-draw
stereo edit through hooked `SetVertexShaderConstantF` for tens of thousands of frames stopped seeing
**any** constant uploads after the game's first `IDirect3DDevice9::Reset`, and never saw one again for
the life of the process. Everything else kept working: `Present` fired at a clean 60 fps, the proxy's
own `Reset` hook logged the event, and its forced-window logic re-applied correctly. Only the constants
path went quiet — and nothing on screen said so.

Three details make this worth a section rather than a bug report:

- **Both kinds of reset trigger it.** One came from changing resolution in the options menu; the other
  from an ordinary **checkpoint restart** that nobody asked for. Forcing a fixed window makes resets
  rare, which is why the symptom had gone unexplained for a session — rarity hid it.
- ~~**It is not instantaneous.** One more healthy per-frame summary printed *after* the reset, and only
  the next was dead — so whatever removes the hook runs roughly 120–240 frames later, not inside the
  `Reset` call.~~ **⚠️ WITHDRAWN 2026-09-04, and the withdrawal is the more useful finding.** The
  summary line that produced that figure counts events *since the previous summary*, over a fixed
  frame window. One healthy summary after the reset is therefore exactly what you would see **even if
  the hook died inside `Reset` itself** — the window, not the mechanism, produced the latency
  `[inferred-static 2026-09-04]`. **A periodic aggregate cannot date an event more precisely than its
  own interval**, and reading a timing claim out of one is a measurement error that looks like a
  result. If you need the moment, stamp the moment: that project's next build logs the slot's state
  the instant `Reset` returns. Filed beside [the instrument can be the
  bug](#the-instrument-can-be-the-bug).
- **The original prime suspect is now excluded.** "The engine re-creates its device-side objects onto
  a path the hook no longer covers" fails on three checks: D3D9 vtables are shared per runtime class,
  a second `CreateDevice` would have been logged by the proxy's own hook, and **another slot in the
  same table kept working** — only the one state-setting slot was rewritten `[inferred-static
  2026-09-04]`. That last detail is what points at the real candidate: [a recorded state block restoring the runtime’s own method table](#recording-a-state-block-rewrites-the-devices-method-table--and-your-in-place-vtable-patch-with-it), immediately below.
- **The control is clean.** Same DLL, same machine, minutes apart: a fresh launch reads healthy every
  time. Relaunch is the only recovery known so far.

**The operational rule generalises to every D3D9 hook in the estate**, and the API-level cause is
engine-agnostic even if the exact mechanism turns out to be one engine's: **treat `Reset` as a
lifecycle event your hook must be *proven* to survive, and test it deliberately** — change a
resolution, then trigger an in-game reload — before trusting any stereo observation taken after
either. Put an "armed" counter in the log (here, the number of per-eye edits applied per summary
interval) and **read it before believing a screenshot**; a stereo mod that has been disarmed renders a
perfectly good mono frame, which is the
[silent no-op](#silent-no-ops-verification-that-cannot-see-the-failure) shape once more.

A second finding from the same run is UE3-specific in cause but worth a line here because it shapes
test design on any title: the game asked `CreateDevice` for the **desktop** resolution on every launch
regardless of the resolution stored in its own config, and only a `Reset` ever applied the stored
value. So on that title a matched-resolution backbuffer and a live stereo were, for the moment,
mutually exclusive — and a per-region stereo measurement had to be shown robust to a uniform
downscale (it is: a uniform scale moves every tile equally, so "this patch is not moving with its
neighbours" survives it).

Generalised from [`enslaved-vr/modding-notes/`](https://github.com/TefMeister/enslaved-vr/tree/main/modding-notes)
(`2026-09-03d-reset-kills-the-stereo-and-glancing-water-measures-clean.md`); the full proxy log of the
run that reset is in that repo's `dev-archive/recon/`.

## Recording a state block rewrites the device's method table — and your in-place vtable patch with it

`[reported]` for the mechanism in general; `[hypothesis]` for the one title where it is currently the
leading explanation. **This is the strongest available candidate for the previous section's symptom**,
and it is worth knowing on its own because it explains a whole class of "my hooks died and nothing
said so" on D3D8 and D3D9.

**The mechanism.** `IDirect3DDevice9::BeginStateBlock` puts the runtime into recording mode by
swapping the device's **state-setting** methods — `SetRenderState`, `SetTexture`,
`SetVertexShaderConstantF` and their neighbours — for recording variants, and `EndStateBlock` writes
the runtime's **own originals** back. Any third-party function pointer sitting in one of those slots
is overwritten by that restore and never comes back. Methods that do not set state — `Present`,
`Reset`, the creation calls — are untouched.

**So the signature is unmistakable once you know it:** *some* of your hooks keep working forever and
*others* die permanently, in the same table, with no error, no crash and nothing on screen. A proxy
that patched five slots finds it still owns four. That is not a partial failure of your patching
code; it is the runtime restoring a specific subset.

**Two independent public witnesses, one on D3D9 and one on D3D8:**

- **gho**, the author of **DxWnd**, on SourceForge (2014-06-02), diagnosing exactly this while chasing
  D3D9 device-`Reset` trouble in a shipped game: *"D3DDevice9::BeginStateBlock recover all COM method
  pointers invalidating the hook patching."* — and the fix he then shipped: *"It's sufficient to hook
  this method to restore back the DxWnd routines and the trick is done!"*
  ([thread](https://sourceforge.net/p/dxwnd/discussion/general/thread/9b1c8171/); read twice,
  independently, before quoting here).
- **Paul Roussin**, on the Microsoft DirectX graphics newsgroup, answering someone whose D3D8 vtable
  hook had failed: *"If you are going to hook the D3D device table that way then you will have to hook
  calls like BeginStateBlock and EndStateBlock. BeginStateblock will reset the device table so you
  have to make the code return control back to you so you can reset your modified addresses."*
  ([archived thread](https://microsoft.public.win32.programmer.directx.graphics.narkive.com/PbJcO31s/hooking-d3device8-by-replacing-the-vtable-fails-info-needed);
  Microsoft's own newsgroups are long gone, so this survives only on a third-party Usenet mirror and
  the displayed date could not be corroborated — treat it as a findable archived post, not a primary
  vendor source.)

**Why it presents as "the reset killed my hook".** You are usually not the one recording. The caller
is another resident of the process — a Steam or driver overlay, anything built on `ID3DXSprite` or
`ID3DXFont`, an engine that records state blocks of its own — and those residents most often
(re)initialise **after a device reset**, which is precisely when the symptom appears. The reset is
the occasion, not the cause. On the worked title the engine's own renderer records no state blocks at
all, so the recorder must be a third party.

**The remedies, cheapest first:**

1. **Detect it.** Every `Present`, compare each patched slot against your own function pointers and
   log the first mismatch with the new value and the module it belongs to. This costs nothing and
   turns an invisible failure into a line in a log.
2. **Re-arm after the fact.** Hook `BeginStateBlock`/`EndStateBlock` and re-apply your patches when
   recording ends — what DxWnd shipped. **Re-arm only a slot that has reverted to the runtime's own
   original pointer**, which is the one unambiguous case; a *foreign* pointer may be a later hook
   that chains to you, and blindly overwriting it breaks somebody else's mod. Log those once and
   leave them alone.
3. **Do not patch the table at all for state-setting methods.** A code hook on the runtime function
   body (MinHook-style) is immune by construction, because nothing about state-block recording
   rewrites the function's first instructions.
4. **Wrap the device** in your own object with your own vtable once you need more than a few methods.
   Wrappers are immune to this entirely — the runtime restores *its* table, and yours is not it.

**Exposure is a one-line grep of your own estate**: any proxy that writes function pointers into a
D3D8/D3D9 device vtable in place is exposed; any proxy that wraps the device object is not
`[inferred-static 2026-09-04]`.

Generalised from a `/gr` research hand-off on
[`enslaved-vr`](https://github.com/TefMeister/enslaved-vr) (2026-09-04), where the self-healing build
described in remedy 2 is written and compiled but **not yet run** — so the mechanism's confirmation on
that title is still owed, and the tags above say so.


## A proxy that PATCHES a vtable slot can be beaten to it — and on Steam it usually is

`[verified-live 2026-09-08, n=2 launches, 4 proxy loads]` Generalised out of
[`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr).

This is the companion to *"a proxy must free the real DLL on detach"* above and to *"Recording a
state block rewrites the device's method table"* — the same family seen from a third side. That one
is about being **reloaded past**; this one is about being **hooked past**. A reader who needs one
almost certainly needs the others.

A `d3d9.dll` proxy installed its hook by writing into **`IDirect3D9` vtable slot 16
(`CreateDevice`)**. Across four proxy loads, that slot was **never free**:

| launch | load | owner of slot 16 |
| --- | --- | --- |
| via Steam | 1, 2 | `Steam\gameoverlayrenderer.dll` |
| **direct exe** | 1 | `Steam\gameoverlayrenderer.dll` |
| **direct exe** | 2 | `C:\Windows\SYSTEM32\apphelp.dll` |

Two things transfer immediately:

1. **Launching the exe directly does NOT avoid the Steam overlay.** Setting `SteamAppId` in the
   environment and starting the exe yourself, bypassing the Steam launcher entirely, still had
   `gameoverlayrenderer.dll` owning the slot on the first load `[verified-live 2026-09-08, n=1]`.
   With the Steam client running, the overlay is injected regardless. If you have been assuming a
   direct launch gives you a clean process, it does not — and note this sharpens, rather than
   contradicts, *"Launching a Steamworks game directly"* above: the launcher can be bypassed, the
   overlay cannot.
2. **The slot's owner changed between two loads 350 ms apart in the same process**, with different
   pointer values — overlay, then `apphelp.dll`. A vtable is shared per class, so something rewrote
   it in between. **Why is not established.** Candidates: a late compatibility shim, the overlay
   re-hooking, or a pointer left dangling by the proxy's own unload/reload that now resolves inside
   another module. Recorded unresolved rather than guessed.

### Why this is worse than it first looks

The same project had already been bitten by the *other* side of this coin. On 2026-09-05, chaining
into whatever was in the slot recursed `CreateDevice` **1,669 times in 1 ms** and killed the process
— the pointer cached as "the real one" was another hook that chains back. A guard was added refusing
to install when the pointer does not live inside the real `d3d9.dll`.

**The guard works, and it is also fatal.** The game runs perfectly — no recursion, clean quits — and
the mod never installs. *"The game runs, this mod does not"* is a correct safety outcome and a
useless product one.

⚠️ And the guard may be **too strict on its own terms**: `apphelp.dll` is the Windows
application-compatibility shim engine, not a third-party hook. If a shimmed `d3d9` legitimately puts
the entry point inside `apphelp`, then "the pointer must be inside the real `d3d9.dll`" is false by
design, and the guard refuses a benign OS mechanism. `[hypothesis]` — consistent with what was seen,
not demonstrated.

### ⭐ The technique to prefer: return your own object, do not patch a shared table

**If you own the DLL export the game calls, do not patch a shared vtable at all.**

A `d3d9.dll` / `dxgi.dll` / `d3d11.dll` proxy *is* the module the game calls `Direct3DCreate9` (or
`CreateDXGIFactory`, or `D3D11CreateDevice`) on. So hand back **your own object** implementing that
interface, forwarding every method you do not care about to the real one. Then:

- nothing is written into a table anyone else shares, so **there is no race to win or lose**;
- other hookers keep working on the real object, layered below you, exactly as they expect;
- it is immune to load order, to overlays, and to OS shims.

It is more code than a three-line vtable patch. **It is the only version that survives an environment
you do not control** — and on Steam, you never control it.

⚠️ **The tempting stopgap is worse than it looks.** Chaining into the foreign pointer with a
re-entrancy guard probably works — but it re-introduces exactly the failure the guard was written to
stop, and that failure is **timing-dependent**: it appeared once in four launches, when the first
block lived 700 ms instead of 16. A fix that is only usually safe, against a bug that is only
sometimes visible, is not worth shipping.

### Cross-check this library has not yet done

The 2026-09-04 sweep read all ten of the account's proxies for the `FreeLibrary` defect. **The same
ten are worth re-reading for this one** — which install by patching a vtable slot, and which return a
wrapper. Any that patch are exposed to this on any Steam title. Recorded here as an open task rather
than a result, because it has not been run.

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

**Add an "imports XInput?" note as you extend this table** — and where the answer is yes, a
**ViGEmBus virtual gamepad is now measured to be the strongest route, not the fallback.**

`[verified-live 2026-09-04, n=2 per axis with reversal]` On a game that imports XInput directly, a
virtual Xbox 360 pad created by ViGEmBus was bound as a genuine controller: the left stick walked the
player as a pure translation with the camera basis unchanged, the right stick turned the view as a
pure yaw with the position unchanged, and each reversed cleanly. The game even raised its
*"Controller Disconnected"* toast when the pad was destroyed on script exit — **positive confirmation
that the game had bound the virtual device**, not merely that Windows had enumerated it, which is the
control most virtual-device tests are missing.

**Three properties make it better than synthetic keyboard or mouse for camera work, not merely
equal:**

- **It is focus-independent.** XInput is polled regardless of which window is foreground, so the whole
  "must be the foreground window" fragility of `SendInput` disappears — and on this very game
  `SendInput` look *does* need focus.
- **It sidesteps exclusive DirectInput entirely.** The trap that swallowed one project's injected
  mouse yaw as exactly 0.0° cannot arise, because the pad is a different device class the game already
  listens to.
- **Nothing is injected.** The OS delivers input through an API the game already calls, so there is no
  hook to install, nothing to crash at startup, and no interaction with a proxy you are also
  debugging.

**The precondition, and the honest limit:** the target must actually import XInput — check the import
table, do not infer it — and the virtual pad must be seen to appear on a slot and round-trip a stick
value before you rely on it. **A game that reads *only* DirectInput for its pad may not see a ViGEm
XInput device**, which is untested here, so treat "imports XInput" as the gate and everything beyond
it as `[hypothesis]`. Where a feature is gated behind a **controller-only** gesture — a thumbstick-click
chord, say — this is the only route that can send it at all.

Generalised from a [`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr) modding hand-off,
2026-09-04; the driver is ViGEmBus, credited in `ATTRIBUTION.md`.

### ⭐⭐ Try the virtual pad FIRST — three games, three engines, and it beat a route that was about to cost days

`[verified-live 2026-09-08, n=2 launches]` Generalised out of
[`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr), with same-day corroboration
from [`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr) and
[`mad-max-vr`](https://github.com/TefMeister/mad-max-vr).

The section above established the virtual pad as the strongest route where a game imports XInput.
A second project has now taken that further, from the opposite direction: **it reached for the pad
only after concluding the game could not be driven at all, and the pad drove it in ten minutes.**

`the-evil-within-vr` had recorded a hard blocker on 2026-09-07 — `SendInput` does not reach the game,
settled across two launches with the controller unplugged and foreground verified before every send;
`Enter` would not dismiss the photosensitivity splash. The queued fix was to port a `GetDeviceState`
injector into the game's proxy: real work, days of it. **It was not needed.** `EvilWithin.exe`
imports `XINPUT1_3`, and a ViGEm virtual pad is that same API with **no code at all**. It drove the
game end to end — the photosensitivity splash *that `SendInput` could not pass*, the attract screen,
the title menu, `CONTINUE` into Chapter 1, the pause menu, and `EXIT` → confirm → clean process exit.
The game raised a *"Controller Connected — Xbox 360 controller"* toast and switched its prompts to
`(A) SELECT / (B) BACK`: it bound the virtual pad as a real one.

**So the order to try things in is:**

1. **Does the exe import `XINPUT1_*`?** If yes, a virtual pad is a ten-minute experiment and it comes
   before writing anything. `pip install vgamepad` plus the ViGEmBus driver, and the pad exists.
2. Only then consider an in-process route (below), and only then a `GetDeviceState` injector.

Two further reasons it should outrank synthetic keyboard/mouse, beyond the three already listed:

- **It is safer than a real controller.** On 2026-09-07 a physical DualSense's **stick drift walked a
  menu highlight from `CONTINUE` onto `NEW GAME`** — a destructive item — while only `Enter`s were
  being sent. A virtual pad's sticks sit at dead centre, so that drift is structurally impossible.
- **A "this game ignores synthetic input" finding is not safe to record until the pad has been
  tried.** That is exactly what happened here: a careful, correctly-measured negative about
  `SendInput` was generalised into "this game cannot be driven", and days of work were queued off it.

#### ⚠️ The trap: the first input after each pad connect is SWALLOWED

Measured on The Evil Within's pause menu `[measured 2026-09-08]`: five `DPAD_DOWN` presses moved the
highlight **three** rows; two presses in a freshly-created pad session moved **zero**. Within one pad
lifetime, after a settle wait, each press moved exactly one row — and the first after connect moved
none.

| step (one pad lifetime) | highlight |
| --- | --- |
| after ~6 s settle | RESTART CHAPTER |
| +1 `DPAD_DOWN` | RESTART CHAPTER — **swallowed** |
| +2 `DPAD_DOWN` | OPTIONS |
| + left stick | TITLE MENU |

A keyboard-derived route saying "Down ×5 to TITLE MENU" would have put `A` on **RESTART CHAPTER**.
Only capture-and-verify caught it. **This is the pad-route equivalent of the silent no-op** — the
input is delivered, the count is simply wrong by one.

**The working pattern:**

- do a whole navigation inside **ONE** pad lifetime; never create a pad per keypress;
- open each session with a throwaway press that **cannot move a vertical list** (`DPAD_RIGHT`) to
  absorb the swallowed input;
- capture and verify the highlight before every commit, always.

##### 2026-09-09 (`/sr`): the driver documents a readiness wait — and it does NOT fully explain this

ViGEm's client API documents an explicit readiness step between plugging a virtual target in and
updating it: plug in, **wait until ready**, then update. Its own wording is *"It may take some time
before the target is ready to accept updates"*, and updating early *"may return `TargetNotReady`
errors"* `[reported 2026-09-09, from the vigem-client API documentation]`. **Any pad route should do
that wait**, and a route that never had one is missing a documented step.

**But read the failure shapes before calling this the cause.** The documented failure is an *error
return* to the caller; the trap above is a **silent** drop — the update reported success and the
input never reached the game. Those are different symptoms, which splits the trap into two
candidates that want different fixes:

| candidate | what it predicts | how to tell |
| --- | --- | --- |
| **driver not ready** | the update call itself fails with `TargetNotReady` | check the return value of every update, including the first |
| **game has not enumerated the device yet** | updates all succeed; the game simply was not listening yet | updates return success and the input is still lost |

The measured case behaved like the second — which is the more awkward one, because no API tells you
when the *game* has finished enumerating a new controller. **So add the readiness wait, and keep the
throwaway-press habit anyway**: the wait removes a real documented failure, and the throwaway press
is the only thing that covers the game-side half. Retiring the habit on the strength of the wait
would be trading a measured protection for a documented one that guards a different thing.

`[hypothesis]` that readiness timing explains any part of the observed drop; `[reported]` only that
the API documents the wait. Credit the **ViGEm** project and the `vigem-client` binding's authors:
<https://docs.rs/vigem-client>.

#### ⚠️ And pad hot-plug toasts can dominate a pixel measurement

Same day, different game: hot-plugging pads mid-session makes Windows draw *"Controller Connected"*
toasts, and those toasts dominated a pixel-difference measurement so badly that they read as the two
strongest "hits" in a button probe — **62× and 85× the control** — while the thing actually being
measured had not moved at all `[measured 2026-09-08]`. If you add pads during a run, either wait the
toasts out before measuring or measure something they cannot perturb. (This is a specific instance of
"Counting events is not measuring content" above.)

### A broken ViGEm bus does not close the pad route — proxy the DLL instead

`[verified-numerically 2026-09-08]` Generalised out of
[`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr) and
[`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr).

The route above needs a working **ViGEmBus driver**. When that driver is broken or absent — as it is
on one machine in this account — the natural conclusion is that the pad route is blocked. **It is
not, and the distinction is worth holding:**

| route | needs | blocked by a broken ViGEm bus? |
| --- | --- | --- |
| ViGEmBus virtual pad | a working bus driver; game reads *any* pad API | **yes** |
| In-process `xinput1_*.dll` proxy | the game importing XInput itself | **no** |

A ViGEm pad is synthesised **for the whole system** at driver level. An `xinput1_3.dll` proxy placed
beside the exe answers the game's own XInput calls **inside the process** — no bus, no driver, no
virtual device, and a broken ViGEm instance is irrelevant. `AliceMadnessReturns.exe` imports
`XINPUT1_3.dll` **by ordinal 2 and 3**, the same shape `prince-of-persia-2008-vr` has and already
ships a loading proxy for (that project needs ordinal 4 as well, so its `.def` is a superset).

⚠️ **This does not demote the ViGEm route.** A virtual pad is seen by games that read DirectInput or
enumerate devices, where a bare XInput proxy is not — the mirror image of the DirectInput caution
above. Two routes, different preconditions; the driver-level one is still the one to try first where
the driver works.

⚠️ **An import is not a call.** The honest tag for a game in this position is **"pad route available,
mechanism untested"** — `prince-of-persia-2008-vr`'s proxy loaded perfectly and that game **never
called `XInputGetState` once**, which is why it now carries an entry-counter instrument. Port the
counter with the proxy and answer it in one launch.

### Check for a game flag that turns the game's own mouse acceleration off

`[reported 2026-09-08]` for the flag; `[measured 2026-09-08]` for the trap it prevents.

An injected mouse delta passes through **two** scalings before it becomes camera rotation: the OS
pointer ballistics, and **the game's own mouse acceleration or smoothing** if it has any. Both are
silent. The injection reports success, the game accepts it, and the camera lands somewhere else — by
an amount that varies with how fast the previous deltas arrived. **A step size calibrated under those
conditions is calibrated against the curve, not the game**, so it does not port between machines and
may not reproduce on one.

**Look for the flag before writing the calibration.** `alan-wake-vr`'s own option table contains
`directaiming`, which Remedy's v1.03 notes describe as removing all mouse acceleration (and enabling
`-rigidcamera` with it); that patch also reworked the low-level mouse reading to cope with low and
variable frame rates, which matters under a VR frame budget. Meanwhile
`alice-madness-returns-vr` hit the OS half the same day and had to measure the machine's ballistics —
thresholds **(6, 10)**, acceleration **ON**, speed 6/20 — precisely because a step size calibrated
there would not port.

**Order:** (1) check for a game flag that disables its own acceleration; (2) only then measure the
OS-side ballistics; (3) record in the project's notes that any committed step size **is valid only
with that flag set** — a number without its conditions is the one that later looks reproducible and
is not. A game reading **Raw Input** is the case where the OS half does not apply, which is another
reason the import table comes first.

⚠️ The general rule is `[hypothesis]`: it rests on one game that ships such a flag and one that hit
the trap, not on a survey. It is cheap enough to try that it does not need to be stronger before
being written down.


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

### ⚠️ "DirectInput ignores injected input" is a pre-Vista folk memory — and it has been costing us the wrong diagnosis

`[reported 2026-09-07, first-party vendor documentation]` Generalised out of the estate's control
profiles.

The belief that a DirectInput game cannot see `SendInput` — because DirectInput "talks to the driver
directly" — is the standard reason given when synthetic input fails against an older title. **It is
wrong on modern Windows.** Microsoft's own DirectInput guidance states that internally **DirectInput
creates a second thread to read `WM_INPUT` data**: its mouse path is a wrapper over Raw Input.
Whatever Raw Input sees, DirectInput sees.

**The real DirectInput trap is on the keyboard side, and it is a different one:** DirectInput reads
**scancodes**, so synthetic keystrokes must carry `KEYEVENTF_SCANCODE` rather than being sent as
virtual-key events. That distinction is the entire reason a separate scancode-based automation library
exists alongside the popular virtual-key one.

**⭐ That mechanism reconciles two contradictory first-hand results in this account**, which is why it
is worth writing down rather than just correcting the folklore. One project found scancodes were the
route that worked and recorded that virtual keys "cost a sibling project a session". Another found the
exact opposite on its own game — scancodes did not reach it at all while the same keys as virtual-key
events worked immediately, contradicting its *own* record from the day before. Both are `n=1`, and both
are consistent with one rule:

> **A DirectInput consumer needs scancodes; a window-message consumer takes either.** So send one, fall
> back to the other, and **record which won, per game**. `[hypothesis]` on that being the whole
> explanation.

> ⚠️ **Superseded in part, 2026-09-07 — read the correction two sections down before relying on this.**
> A controlled test on one title sent scancodes to a `NONEXCLUSIVE`, foreground, 200 Hz-polling
> DirectInput keyboard for 22 seconds and the game saw nothing. The rule is a good **first thing to
> try**; it is not an explanation.

This is the concrete form of the estate's standing rule to
[build several input routes and measure which the game obeys](#injected-input-measure-it-against-a-control-never-against-zero):
the two routes are not redundant, they select for different consumers.


### ❌ Correction, 2026-09-07: a controlled test contradicts the rule above, and the reason is undetermined

`[verified-live 2026-09-07, n=1 game, with controls]` This was published earlier the same day and a
project measured against it within hours. **The counter-example is recorded before any explanation,
because the explanation is genuinely open.**

On one 2008 D3D9 title: game **foreground**; keyboard acquired **`NONEXCLUSIVE`** (the case where
injection would normally be seen); the game polling `GetDeviceState` at roughly **200 Hz**; a key held
down via `SendInput` **as scancodes** — the route the rule above recommends — for **22 seconds**,
spanning four logged samples. The game's own instrumentation read `keys currently down: 0` throughout.
**`SendInput` did not reach that game's DirectInput keyboard state.**

The rule above is therefore **not general**, and this library does not yet know why. Four candidate
explanations, each with the observation that would separate it — **none of them has been run**:

| candidate | what would settle it |
| --- | --- |
| The vendor's "`WM_INPUT` reader thread" statement is about DirectInput's **mouse** high-DPI path specifically, and the **keyboard** path is not a Raw Input wrapper at all | inject a **mouse** delta into the same game and watch its `DIMOUSESTATE`. Mouse through, keyboard not ⇒ the split is per-device, and the rule above should be scoped to the mouse |
| **UIPI**: the harness ran at a different integrity level, and the failure was the silent one described above | compare the two processes' integrity levels; or fire the identical injection at a control application and confirm it lands |
| This title loads a **redistributable or shimmed `dinput8`** rather than the system one, so the modern implementation is not in play | check which module the process actually loaded, and its version |
| Injection **shape or timing** | already weak: the key was held 22 s across four independent samples |

**⚠️ Do not read this as "DirectInput cannot see injected input" either.** That is the folk memory the
section above was written to retire, and one controlled negative on one 2008 console port does not
restore it. What is established is narrower and worth stating exactly: **the vendor documentation does
not license a blanket prediction, and a per-game measurement is still required.** Treat the scancode
rule as *the first thing to try*, not as an explanation of what you observe.

### ⭐⭐ And when no OS route reaches the game: write into the buffer the game asks for

`[verified-live 2026-09-07, n=1 session]` Generalised out of
[`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr).

The project above stopped fighting the input stack and went underneath it. **The device-state call is
already hooked for diagnostics; after the real call returns and before the game sees the buffer, OR in
a state block that the harness writes from outside the process via shared memory.** The game is asking
"what keys are down?" — answer it.

This is worth reaching for **earlier than it usually is**, because it removes every variable the
sections above are about: no scancode-versus-virtual-key question, no pointer ballistics, no UIPI, no
foreground requirement, no dependence on Windows delivering anything at all.

In the worked case that meant the 256-byte keyboard array plus both the 16-byte and 20-byte mouse
structures. **What the hook does with the buffer is where the failures live** — four rules, the
pure-function property that makes them testable without a launch, and the shared-vtable hazard are all
[below](#-the-four-apply-rules-each-named-for-the-failure-it-prevents).

**Four independent readings confirmed it, which is the standard worth copying** rather than the
technique alone: an `applied` counter in the proxy (355 in six seconds of one held key); **the game's
own instrumentation** flipping from `0` to `1` keys down; **by eye**, two injected taps moving a menu
highlight exactly two rows; and a **frame-difference measure of 24.08 against a 0.00–0.23 no-input
baseline measured on the same scene**. The last one is the
[control that turns a positive into evidence](#injected-input-measure-it-against-a-control-never-against-zero),
and the baseline being *near zero* rather than merely *small* is what makes the reading unambiguous.

**⚠️ Two method lessons from the same session, both cheap and both expensive to relearn:**

- **The original injection test had been run at the TITLE SCREEN — the one place in that game that
  polls no input at all.** Every negative from it was worthless: the test could not have gone positive.
  The device-level hook log is what exposed it, showing zero polls at the title screen and ~200 Hz
  during gameplay. **Before trusting an input negative, confirm the game is reading input at that
  moment** — and note that a *console port* is exactly the kind of title whose menus wait on a gamepad
  rather than a keyboard, which is what this one turned out to be doing.
- **The import table settled what could not work by construction.** That executable imports
  `DirectInput8Create` and XInput and **none** of `GetAsyncKeyState`, `GetKeyboardState`, `GetKeyState`,
  raw input, `GetMessageA` or `ToAscii` — only `PeekMessageA`, the pump itself. So the posted-message
  route was excluded **before** a single test was written. See
  [read the import table before you design the input layer](#read-the-import-table-before-you-design-the-input-layer).

#### The transport is the problem — and injecting below it removes the whole class at once

`[verified-live 2026-09-07]` · the rules below `[verified-numerically 2026-09-07, 36 host checks]`

Everything the sections above say about injecting *through* Windows is a list of things that can
silently defeat you. **Every one of them is a property of the transport**, not of the game:

| | inject **through** Windows | inject **inside the read path** |
| --- | --- | --- |
| works without window focus | ✗ — `SendInput` follows focus | ✓ |
| survives an integrity mismatch | ✗ — UIPI fails **silently** | ✓ |
| delta scale is portable | ✗ — ballistics, up to 4× | ✓ — you write the value the game reads |
| a remapper in the middle | ✗ — Steam Input and friends sit in the path | ✓ — irrelevant |
| needs a proxy or hook | ✓ not required | ✗ — **the real cost** |
| proof of success | inferred from behaviour | **the game's own counters change** |

The principle is not about DirectInput or about one game: **find the call where the game asks the OS
for input, and answer it — instead of asking the OS to tell the game.**

#### ⭐ The four apply rules, each named for the failure it prevents

The hook itself is the obvious part. What the hook *does* is where the failures live:

1. **One-shot relative motion.** A delta left standing in the block is re-served on every poll — a
   camera that spins and never stops. **Make the write consume itself.**
2. **OR semantics, never replace.** A physically held key must not be cleared by the injector, or
   synthetic and human input fight each other.
3. **Discriminate by state size, and REFUSE unrecognised sizes.** A permissive "big enough" check is
   how the worst input bug in this account happened — see the vtable pair below.
4. **⭐ Bit-for-bit no-op when disabled, or on bad magic.** The underrated one: it lets the hook stay
   **permanently installed and be *proved* inert**, so *"is the hook itself the problem?"* becomes
   answerable without uninstalling and relaunching. This is
   [the instrument can be the bug](#the-instrument-can-be-the-bug) solved by construction rather than
   by a control run.

**⭐⭐ And the shape matters as much as the rules: the apply step is a pure function of
`(buffer, size, desired state)`, so it is unit-testable on the host with no game and no launch.** One
implementation carries 36 host checks passing with nothing running. For a technique whose failures
otherwise cost live sessions at the most expensive gate available, that is the single most valuable
property it has — design for it deliberately, and keep the impure parts (hook installation, shared
memory) outside the function under test.

#### 🚨 The shared-vtable pair — two opposite failures, one cause

`[verified-live]` on both halves, from two different projects. **DirectInput devices of the same class
share one vtable**, and that single fact produces two failure modes that look nothing like each other:

- **Patch via one device, and your hook fires for the other.** One project hooked through the *mouse*
  and found its handler running for the *keyboard*: mouse deltas landed in the key-state array,
  index 1 being `DIK_ESCAPE`, so a pause menu opened "by itself" — and **silently invalidated three
  experiments** before anyone noticed. Fix: record the device **instance** pointer and require
  `device == that pointer`.
- **Register only the first device, and instrument the wrong one.** Another project registered just
  the first device created; that game creates the **mouse** first, so its keyboard was never
  instrumented and the log looked devoid of activity — a convincing false negative. Fix: register
  **every** device and store originals **per vtable** rather than in single globals.

**The two fixes are complementary, not alternatives**, and each looks like the whole answer when read
alone: one narrows *what you act on*, the other widens *what you observe*. Take both. And note that
this is exactly why rule 3 above says **refuse** an unrecognised state size rather than tolerating it
— when one vtable serves several device classes, buffer size is the only thing distinguishing them at
the call, and a permissive check is what let the mouse deltas into the key array.

Credit our own `prince-of-persia-2008-vr` (the injector, the hook-every-device fix and the host test
suite) and `ai-game-control-profiles/UNIVERSAL.md` (the shared-vtable rule and the incident behind it).
No public source was involved in this finding.

**⚠️ Confidence, stated at the scope it earns:** the live result is `[verified-live 2026-09-07]` on
**one game**; the apply rules are `[verified-numerically]` as host tests and `[inferred-static]` as
*general* rules — they are one implementation's answers, not a survey; and the generalisation to other
engines and input APIs is `[hypothesis]`, filed because the reasoning is transport-independent and the
cost of trying is a proxy most of these projects already ship, **not** because it has been demonstrated
twice.
### Three documented ways a game *could* filter injected input — with their OS-version floors

Worth knowing so a future negative can be diagnosed rather than guessed at:

| API | what it exposes | floor |
| --- | --- | --- |
| `MSLLHOOKSTRUCT.flags` | `LLMHF_INJECTED`, `LLMHF_LOWER_IL_INJECTED` — visible **only to a low-level hook**, not to ordinary message handling | long-standing |
| `GetCurrentInputMessageSource` | `originId == IMO_INJECTED` for `SendInput` from a non-UIAccess process | **Windows 8+**, so an older engine *cannot* be using it |
| `GetMessageExtraInfo` | the injector's own `dwExtraInfo` tag | long-standing |

The OS floor is the useful column: if the game predates Windows 8 it cannot be using the middle row,
which removes a whole class of "the game detects us" theories without any experiment.

### ⚠️ Two rules that belong beside every `SendInput` in this library

- **UIPI failure is SILENT.** Input may only be injected into a process at an equal or lesser
  integrity level, and when it is blocked **neither the return value nor `GetLastError` reports it**
  `[reported]`. A harness must run at the same integrity level as the game, and **"no effect" must not
  be read as "the game ignores injected input" until that has been checked.** This is a
  [silent no-op](#silent-no-ops-verification-that-cannot-see-the-failure) sitting underneath every
  input experiment.
- **⭐ An injected mouse `dx` is not a portable unit.** Windows pointer ballistics scale injected mouse
  deltas by **up to 4×** depending on the pointer speed and threshold settings `[reported, vendor
  documentation]`. A figure measured on one machine — "120 steps of `dx=40`" — is a property of *that
  machine's* pointer settings as much as of the game, and this account came close to copying one to a
  sibling project as though it were an engine constant. Pin the values via `SystemParametersInfo` at
  harness start, or calibrate against a read-back.

### And one thing that is genuinely unresolved — recorded as unresolved

**Whether `SendInput` reaches a pure Raw Input consumer is contested in public sources**, and this
library takes no side:

- **Against:** remote-desktop and game-streaming projects report their `SendInput` path producing
  absolute packets or zero deltas in raw-input games and needing kernel HID injection instead; one
  states that the Windows cursor may move while a game listening only for Raw Input receives nothing.
- **For:** the entire user-mode injection ecosystem targets raw-input shooters, and the anti-cheat
  literature treats user-mode injection as *working but detectable* — which is only coherent if the
  events arrive.

Plausible reconciliation: the streaming failures are about **injection shape** — absolute coordinates,
or relative deltas defeated by the game's own cursor clamping — rather than a hard OS rule
`[hypothesis]`. Either way, **a null result against a raw-input game is not self-explanatory** and
needs a control before it becomes a conclusion.

Credit **Microsoft Learn** for the DirectInput high-DPI mouse guidance (the `WM_INPUT` thread),
`SendInput`, `MOUSEINPUT` pointer ballistics, `MSLLHOOKSTRUCT`, `GetCurrentInputMessageSource` and
UIPI; **learncodebygaming** for `pydirectinput` and the scancode requirement; **changeofpace**
(`MouClassInputInjection`) for the injected-flag observation; and **ClassicOldSong** (Apollo) and the
**LizardByte / Sunshine** team for the raw-input failure reports. All read online; nothing cloned or
copied. The two contradictory first-hand results being reconciled are our own, in the
`enslaved-vr`, `alan-wake-vr`, `doom-2016-vr` and `psychonauts-vr` control profiles.

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

### 1b. The static-search version: three controls turn an exhaustive negative into evidence

`[verified-numerically 2026-09-05, n=2,464 blocks, 0 hits]` Everything in §1 is about live tests. The
same discipline has a *static* form, and it is what separates a searched-and-not-found result that
closes a line of work from one that merely records an absence of skill.

The case: a game's debug menus were believed to be wired up through a particular kind of data block,
of which the shipped data contains 2,464 across 18 archives (9.28 GB decompressed). The search found
**zero** references to either debug-menu id, in either byte order, and none to any of the 19 ordinary
menu ids used as controls. That negative is now strong enough to close the hypothesis and be written
into the dead-ends list — but only because three separate controls were run alongside it, and each
one rules out a different way of being wrong:

1. **A population control — does an independent count agree?** The 2,464 figure matched a
   type census produced three days earlier by unrelated tooling, exactly. This rules out *"the
   extractor is missing most of the data"*, which is the failure that makes an exhaustive search
   non-exhaustive without ever saying so.
2. **A detector control — does the scan find the class of thing it is looking for?** 99.0% of those
   blocks (2,439 of 2,464) were shown by the *same* scan to carry other blocks' raw ids. So the
   detector demonstrably finds embedded ids; it simply did not find these. This rules out *"the
   matcher is broken"*.
3. **An encoding control — could the thing be written a way you did not search?** The ids were
   searched as little-endian words, as big-endian words, **and** as ASCII text — and the text sweep
   found two other known ids 274 and 2,390 times respectively. This rules out *"it is there in a
   representation I never looked for"*, which on a game's data files is the single likeliest way to
   manufacture a false negative.

**And the negative earned its keep by being paired with a structural reason.** Decoding the 39 menu
handler blocks showed each is nothing but *{own id, type hash, UI-file id, name length, screen
name}* — a binding of a name to a UI file, **with no field capable of holding an action list at
all**. A negative search says *"not found here"*; a layout says *"could not be here"*. When you can
get both, the conclusion stops depending on the quality of your search, and the hypothesis is
genuinely closed rather than merely unproven.

The cost of skipping this is not the wasted search — it is that an unqualified "we searched and
found nothing" gets re-searched by the next session, or worse, believed. Generalised from
[`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr), 2026-09-05.

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

## Prove an effect by REVERSING it, not by scoring it

Two unrelated projects landed on the same discrimination on the same day, one in rendering and one in
input, and it is worth stating on its own because the alternative — a similarity score between two
screenshots — has repeatedly failed on both.

**The rule: an effect is real when pushing it one way moves the thing, pushing it the other way moves
it the other way, and restoring the parameter restores the original exactly.** Scene animation, idle
drift, autoexposure, a breathing camera and a wandering NPC can all produce a difference score. None
of them can produce a **proportional, signed, reversible** response to a parameter you control.

- **In rendering** `[verified-live 2026-09-04]`: a stereo shear at default parameters was
  *sub-visible*, and the frame difference it produced (mean ≈ 5, no coherent horizontal shift) was
  indistinguishable from animation. Saturating the two parameters slid the whole scene bodily
  sideways; restoring them recentred it exactly. Proportional, horizontal rather than vertical, and
  reversible — three properties animation cannot fake, and together they promoted an inconclusive
  session to a settled result. This is
  [saturate first, then tune down](#saturate-first-then-tune-down--a-too-small-injection-reads-exactly-like-failure)
  arriving from the rendering side.
- **In input** `[verified-live 2026-09-04, n=2 per axis with reversal]`: each stick axis of a virtual
  pad was confirmed by an **isolated** motion read off the engine's own camera values — a pure
  translation with the camera basis unchanged, then a pure yaw with the position unchanged — and then
  reversed. The project explicitly abandoned luma-difference scoring for this, having been misled by
  it before.

**The practical form:** pick a readout that is a *number the engine owns* rather than a picture where
possible; move the parameter far enough to be unmistakable; then move it back and check you land on
the original value. Where only a picture is available, demand that the change be **directional and
proportional**, not merely present. And record the reversal in the log — a result that was proven by
reversing should say so, because the next reader's first question is whether scene noise could have
produced it.

Generalised from [`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr)
and [`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr), both 2026-09-04.

### And the sweep that changes *nothing* is a positive result about where the cause is not

`[verified-live 2026-09-04, n=2 outdoor locations]` The mirror image of the rule above, and worth
naming because it is routinely thrown away as a failed experiment.

A compositor was blowing snow to flat white. Exposure was the obvious suspect, so it was swept across
a **20× range** — and the output did not change. That looks like a wasted afternoon. It is not: a
parameter that moves 20× while the symptom sits still is **not the parameter driving the symptom**,
and that is a conclusion, obtained without needing to understand the thing that *is*.

Two supporting reads then localised it in minutes rather than hours. A probe showed the *source*
values were finite and unremarkable (well inside range) while the compositor's own output peaked
below one — so the saturation was being introduced between the two, not inherited from the input.
Switching off one optional processing package made the snow go dark immediately. Cause located,
package retired, and the problem it had originally been built to solve turned out to be already
solved by a simpler change made for other reasons.

**Three habits fall out of it:**

- **Sweep wide before sweeping fine.** A 20× sweep answers *"is this the knob?"*; a 10% sweep only
  ever answers *"what does this knob do near here"*, and if the answer is "nothing" you cannot tell
  a wrong knob from a small effect.
- **Bracket the pipeline, not just the output.** One reading at the input and one at the output of
  the stage you suspect converts *"it looks wrong"* into *"it goes wrong between here and here"*.
- **Say plainly when you have not explained it.** The write-up records that *why* the offending
  package reached that content is still not understood, tagged as a hypothesis, rather than
  inventing a mechanism to go with a working fix. Removing a symptom is not explaining it: a fix
  that also stops the failing path from being exercised has proved nothing about the cause, however
  well it works.

Generalised from [`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr),
2026-09-04.

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

### 4. A synthetic tap has a minimum length, and it is per machine

`[verified-live 2026-09-04]` The same harness (Win32 `SendInput`, scancodes, no virtual keys) that
had driven a game's menus with **70 ms** taps on one machine was **silently ignored by the same game
on another** — two `Esc` presses, screen unchanged, the game rendering normally and holding focus.
**250–300 ms holds registered every time**, across about twenty presses. Nothing in between was
tested, so the threshold itself is unmeasured; what is established is that the working length is not
a constant of the game.

The likely cause is ordinary and unfixable from outside: an engine that samples input once per frame,
or debounces it, needs the key down across at least one sample, and frame time differs per machine
and per resolution. **The point is the failure shape, not the mechanism** — a too-short tap and an
unbound key are indistinguishable, and both look like "this game ignores the keyboard", which sends
you to rewrite an input backend that works. Same family as the extended-key flag on the arrow keys,
and as [a too-small injected mouse motion](#saturate-first-then-tune-down--a-too-small-injection-reads-exactly-like-failure):
the stimulus was below threshold, not absent.

**Rule: before concluding a binding is wrong, lengthen the hold to about 0.3 s.** Then record the
working hold length in that game's control profile, **per machine** — it is machine state, not game
knowledge.

### 5. Read the game's own keymap file before guessing which key does anything

Guessing keys is the most expensive habit in this area, and community-reported keys are only slightly
cheaper: they are usually right for a different build or a different layout. **Many games ship their
bindings in plain text** and will simply tell you.

The worked case stores its actions in an ini as **alphabetical indices** — `A`=0 through `Z`=25 — so a
row reading `12` is `M`. Decoding that file named the first-person-driving key and the enter-vehicle
key before either had been pressed `[inferred-static 2026-09-04]`. **Config-backed beats
community-reported, and both beat guessing** — and a config-backed key that then does nothing is a
much more informative negative, because the binding is no longer in doubt.

Carry the standing caveat with it: [a binding surviving in a shipped config is not evidence the
feature is live](#and-check-that-the-shipped-switch-still-dispatches--a-binding-in-a-config-is-a-lead-not-a-feature).
The file tells you which key, not whether anything is listening.

### ⭐⭐ 6. A console that can `exec` a FILE is a full scripting channel over one keypress

`[reported 2026-09-07]` Generalised out of
[`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr).

Everything above is about getting *characters* into a console reliably, which is fiddly: layouts, dead
keys, tap lengths, scancode-versus-virtual-key. **There is often a way to type almost nothing at all.**

Most engine consoles have a command that executes a file of commands. Bind **one key** to
`exec <file>`, put that file where the engine looks for it, and then **rewrite the file from your
harness between presses**. The next press runs whatever you just wrote. One synthetic keypress —
already the most reliable thing in the input layer — becomes an arbitrary command channel, and the
whole character-entry problem disappears.

Two details that make or break it in practice:

- **The engine may want the file extensionless**, or in a specific directory. That is a per-engine
  fact to establish once and record in the dossier.
- **Rewrite the file, do not append.** The press executes the current contents; a stale line left at
  the top will run again every time.

**⭐ And once you have that channel, look for an ABSOLUTE pose-setter before writing any relative
input.** Engines with a debug/cheat console frequently ship one — the worked case is UE3's
`BugItGo <X> <Y> <Z> <Pitch> <Yaw> <Roll>`, which sets the player's location **and rotation** outright,
with a companion command that prints the current pair. That changes the character of the whole
problem:

| | injected relative input | absolute pose command |
| --- | --- | --- |
| repeatability | depends on sensitivity, ballistics, frame timing | exact |
| verification | measure the result and hope | **print the pose back and compare** |
| calibration | required, per machine | none |

**A camera you can set and read back is self-verifying**, which retires an entire class of "did the
input land, and by how much?" experiments — including the
[pointer-ballistics portability trap](#-directinput-ignores-injected-input-is-a-pre-vista-folk-memory--and-it-has-been-costing-us-the-wrong-diagnosis)
above, since there is no `dx` to scale.

**So the order of investigation is: shipped console → `exec` file channel → absolute pose command →
and only then injected mouse movement.** One project had a mouse-injection row on its board and found
two keyboard-only routes sitting in front of it. ⚠️ Check the command *names* against the engine
version rather than assuming them — two plausible-sounding ones in that same investigation do not
exist under the names first tried, which is the ordinary case for console vocabulary.

Items 4 and 5 generalised from `mad-max-vr` modding-session hand-offs, 2026-09-04; item 6 from
`alice-madness-returns-vr`, 2026-09-07.

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

### A photo mode is also a free testbed for the camera and projection *constants*

The section above is about what a shipped photo mode tells you. This one is about using it as an
instrument, which turned out to be worth more.

**First, check that the photo camera writes the same constants gameplay writes.** On one D3D11
title the pause-menu capture camera drives **exactly the shared constant-buffer slots the gameplay
camera drives** — the camera-position slot and the per-pass clip matrix `[verified-live 2026-09-04,
n=1 game]`. That is not guaranteed; a photo mode could plausibly run its own path. The test is one
launch: dump the slots in gameplay, enter the photo mode, move the camera a known way, dump again,
and see whether the same slots moved by the expected vector.

Where it holds, you have gained the cheapest per-eye rig available: **the scene is frozen, the
camera is still and steerable, the HUD is gone, and an A/B of a rewritten constant is a screenshot
pair with no timing pressure.** Animation noise — which otherwise contaminates every two-dump
comparison — is simply absent.

**Second, an FOV slider hands you the projection at several known angles, for free.** On the same
title the slider moved **only the two focal columns** of the shared view-projection, sweeping
horizontal FOV from 58° to 117° while the eye position and forward vector stayed put
`[measured 2026-09-04, n=6 dumps, 5 slider positions]`. For anyone about to write a per-eye
projection rewrite that is a built-in reference implementation: set a known angle, read the columns,
compare against your own maths. It also localises the projection inside a fused matrix without a
single memory write.

**Third, two window aspects tell you which FOV the engine anchors.** Running the same slider value
at 16:9 and at 1.40:1 gave the same horizontal FOV and a different vertical one — so that engine
anchors the horizontal and derives the vertical `[measured 2026-09-04, n=2 aspects]`. One extra
launch answers it for any game, and it decides which column a VR patch must scale.

**Fourth — the practical trap — photo-mode UIs are often mouse-only.** Two sessions on that game
spent keypresses on `E`, `Tab` and the arrow keys trying to change tab; a mouse click on the tab
label worked first time, and slider values moved only on clicks to the `<` / `>` arrows at the ends
of the bar — not bar clicks, not knob drags `[verified-live 2026-09-04]`. **When an in-game UI
shows a mouse glyph in its hint row, drive it with absolute clicks before spending any more
keypresses**, and record the working click coordinates with their window size, since they are
resolution-dependent.

Generalised from `mad-max-vr` modding-session hand-offs, 2026-09-04
([notes](https://github.com/TefMeister/mad-max-vr/tree/main/modding-notes)).

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

### …and check that the shipped switch still *dispatches* — a binding in a config is a lead, not a feature

`[verified-live 2026-09-03, n=6 keys + 1 controller chord, one build]` The habit above has a failure
mode of its own: a shipping build can keep every **binding** and strip the **dispatch** the bindings
call. Enslaved (UE3) ships console key bindings with no console, a full debug-camera input map with
no reachable debug camera, and a *developer's own* `F9 = shot` binding in a shipped ini — and none of
them do anything, because console-style exec commands (`FOV`, `shot`, `ToggleDebugCamera`) dispatch
nowhere in that build. The controller chord that would open the debug camera also did nothing, for
the same one reason: its job is to *call* an exec command, and exec dispatch is gone. Movement, menus
and a hot-plugged virtual XInput pad all worked, so the input side was fine; the far end was missing.

Two things to carry:

- **Config is a lead; running it is the evidence.** A binding surviving in a shipped ini says the
  feature existed when the ini was authored, not that the build can reach it. Six keys across two
  sessions is enough to stop trying keys and go looking for the dispatch itself.
- **Build the test so it can go positive.** The `F9 = shot` case was chosen because it ships in the
  developer's own ini and does not depend on any edit of ours — so a null there is a real null. The
  positive control for the pad was the same pad moving the character a moment later.

When the dispatch is stripped, the remaining command channels are in-process (call the engine's exec
entry point directly from your own proxy, located by pattern — static work) or a virtual controller
for anything gated behind a pad; the estate's proven virtual-pad tool drove this game hot-plugged,
with no restart.

Generalised from [`enslaved-vr`](https://github.com/TefMeister/enslaved-vr), 2026-09-03.

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

### ⚠️ A cross-reference scanner that does not decode ModRM is blind on x64 — and every "no xrefs" result it produced is suspect

`[verified-numerically 2026-09-05]` The worst member of this family found so far, because the tool is
**correct on one architecture and near-useless on the other**, and nothing in its output says which
one you are on.

This account's own `static-disasm.py` (in
[`flat-to-vr-RE-toolkit`](https://github.com/TefMeister/flat-to-vr-RE-toolkit)) finds references to an
address by matching exactly two things: the address as a whole little-endian pointer anywhere in the
image, and `E8`/`E9` rel32 call/jump displacements in executable sections. It never decodes ModRM, so
**it cannot see a RIP-relative operand** — `lea rax, [rip+disp32]`, `mov rax, [rip+disp32]` and the
rest of that family.

**Why that is an architecture-shaped hole rather than a bug.** On x86-32 the compiler addresses data
by absolute immediate, so a whole-pointer scan catches nearly every data reference and an empty result
is decent evidence of absence. On x64 the compiler emits **RIP-relative addressing for most data
references**, and an absolute pointer survives only where something stored one in memory. The same
command that was near-conclusive on a 32-bit target is close to meaningless as a negative on a 64-bit
one — and this account's estate splits roughly in half, with both halves having used the same command.

**The consequence for anything already written down.** Any past conclusion of the shape *"X is not
referenced anywhere, so it is dead / generic / unused"*, drawn from a cross-reference scan of a 64-bit
image, **is not the negative it looks like** and should be re-checked before it is relied on. No such
audit has been run; this is a flag over a class of claims, not a finding about any one of them.

**What closes the gap.** A scanner that decodes the RIP-relative forms and resolves
`next_instruction + disp32`. One exists and is committed — `riprefs.py`, in
[`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr)'s
`dev-archive/recon/2026-09-05-reflection-eye-field-hunt/tools/` — though at the time of writing it has
been exercised against exactly one image, so treat it as working rather than hardened. Folding it into
the shared tool as a mode was deliberately *not* done in the session that found the hole: it changes a
shared tool's interface on the strength of `n=1`. The interim mitigation shipped instead is worth
copying — the tool now **prints a warning on the zero-hit path only when the image is 64-bit**, so the
one output that can lie says so, and the 32-bit case stays quiet.

**The generalisable habit, which is the reason this is here rather than in one project's notes.** The
hole was not found by auditing the tool. It was found because a project noticed that a table it
believed was referenced reported *zero* references, wrote a second scanner to check, and only then
discovered the first could not have seen them either way. **When a negative is load-bearing,
re-derive it with an independently written detector.** Note the outcome: the second scanner *agreed*
— the tables genuinely have no code references — so the conclusion survived while the reasoning
behind it did not. That is the ordinary result, and it is still worth the hour, because the next
negative from the same command would not have survived.

Generalised from a `/pd` hand-off out of [`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr),
2026-09-05.

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

### ⚠️ "All seven candidate accessors are absent" describes your guess, not the object

`[verified-live 2026-09-05, n=2]` Generalised out of
[`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr).

A reflection API tempts you to probe an unknown object by **name**: try `get_Resource`, then
`getResource`, then `get_Handle`, then four more, and conclude the object exposes nothing useful. One
project did exactly that against a texture wrapper, recorded "all seven candidate accessors absent", and
built a design decision on it. The result was true of the seven names guessed and said nothing whatever
about the object.

**Enumerate the type's own members instead.** Every reflection system that can answer "does this method
exist" can also answer "what methods does this type have" — walk the zero-argument value getters and
report what is actually there. The rewritten probe does this, and it is the difference between a negative
about the object and a negative about your vocabulary. This is the same shape as
[a cross-reference scanner that does not decode ModRM](#-a-cross-reference-scanner-that-does-not-decode-modrm-is-blind-on-x64--and-every-no-xrefs-result-it-produced-is-suspect):
an instrument whose blind spot is invisible in its output.

**Three guards for the enumeration itself:**

- **Call only value-type and string returns.** A primitive getter is a field read; an object getter may
  **construct** something, with side effects inside someone else's frame.
- **A scripting-language `pcall` does not catch an access violation.** A guarded call is not a safety net
  for a bad pointer — the process dies anyway. Restricting *what* you call is the actual protection.
- **Read one known object last, as a control.** That project reads a resource whose dimensions and format
  were established a fortnight earlier; a wrong reading there voids every negative in the same run. It is
  the habit that had already caught three wrong answers the same day.

**One more failure of name-based probing, opposite in shape:** in the same session,
`find_type_definition("via.render.RenderTargetTextureResource")` returned nil while the resource
**factory resolved the identical string** and handed back a working object. Resource types were simply not
managed types on that build. When a name lookup fails but something else accepts the name, **take the type
from the returned object**, not from the lookup that failed.

### 🚨 …and the false POSITIVE: never name the string you are asking a fetcher to find

`[measured 2026-09-07, n=1 page, 2 fetches]` on the incident · `[hypothesis]` on the frequency.
Generalised out of [`visceral-re2-vr`](https://github.com/TefMeister/visceral-re2-vr).

Every row above is a fabricated **negative**. This one runs the other way, and that makes it worse: a
fabricated **positive** enters the record as a fact and gets acted on later.

During material research, a page fetch was asked, in effect, *"does this forum thread mention
`Detail_UVScale`, `DetailNormalMap`, `DetailMaskMap`, `DetailIntensity`, `DetailNormalPower`?"* The
fetcher returned **a quote listing those five names as though they were the thread's content**. A
neutral re-fetch of the same URL showed the thread contains none of them — it is five short posts
about a broken download link.

**The dangerous part is that this is what the standard defence looks like when applied naively.** The
rule against fabricated negatives says to prove the fetch *could* have returned a positive by putting
a known-present item in the query. Name the string you are actually testing for, and you have handed
the summarising model the answer to give back. **The two rules pull against each other, and the
resolution has to be stated:**

- **To prove the fetch is capable of a positive,** use a control string you **already know** is on
  that page — never the string whose presence is the question.
- **To test whether a string is present,** ask an **open** question — *"list the parameter names this
  page mentions"*, *"what does this thread discuss?"* — and then look for your term in the answer.
  Never *"does this page mention X?"*

**This is the second confirmed case in this account of an automated read producing text a page does
not contain.** The first was a wiki serving cloaked "AI instructions" to fetchers rather than to
browsers — a hostile page. This one is a **compliant summariser**. Same failure surface, opposite
cause, and only the second one is invisible to anyone reading the page by hand afterwards.

**The cheap habit that caught it:** a second, differently-worded fetch of the same URL. Where a claim
is about to be written down as fact, re-derive it once with a prompt shaped differently from the
first. If the two disagree, neither is evidence yet.

**⚠️ Two days later this rule caught this library itself, and the way it failed sharpens it.** A claim
about one mod's configuration was published here after a verification pass — and **the verification
question named the string it was verifying**, asking the fetcher to quote any sentence about the exact
feature under test. It produced one. The clause survived into a curated entry and had to be withdrawn.
So:

- **The rule binds VERIFICATION fetches, not only discovery fetches.** A confirming question is the
  *easiest* one to answer wrongly, because it hands over the shape of the answer. The check that is
  supposed to catch a fabrication is the one most likely to reproduce it. Verify with an open prompt —
  *"summarise this thread and quote the developer's replies verbatim"* — and read the answer for your
  term.
- **⭐ A claim that appears only in summarizer prose, and never in the body of a page actually fetched,
  is not `[reported]`. It has no source yet.** That is a mechanical test, it needs no judgement, and it
  would have caught this before publication. Apply it before promoting anything to a curated file.

**And note what an honest empty result looks like**, since the fear of a fabricated negative pushes the
other way. When the same source was later asked openly for its roadmap, it returned material no
summarizer would invent — asymmetric frustum handling, shadow maps shared between eyes, specific OpenXR
interaction profiles — and repeated pagination returned the same set. **Content too specific to
confabulate, plus a stable result across pages, is what "the fetch really read the page and the thing
is not there" looks like.**

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

### A recognizer is only as specific as the measurements it takes — and tightening it can refuse the case the design depends on

`[verified-live 2026-09-05, n=7 latches]` Generalised out of
[`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr).

The section above says to identify a resource by how it is used. When you cannot — when you must latch
onto an allocation at creation time because that is the only moment it is visible — the predicate you
write is a **hypothesis about what makes this resource unique**, and it silently expires.

One project's predicate checked dimension, width, a height window and one usage flag. It worked for two
weeks, and then the engine allocated an HDR intermediate at the same size, which passed all four, and the
picture went black. **Width plus height plus one flag was enough on the day it was written and stopped
being enough the moment the engine allocated something else the same size.** The fix is not cleverness but
census: log every allocation the predicate accepts and every near miss, and check what actually
distinguishes the one you want. Here it was the format and one flag — every genuine target had arrived as
`R8G8B8A8_UNORM_SRGB` with no unordered-access flag across four latches in two sessions.

**⚠️ And then the obvious fix was a regression, which is the more useful half of the story.** The format
gate was added to the shared predicate — and the design *depends* on later accepting a different format:
the pipeline latches an 8-bit resolve first and then **upgrades** to the raw HDR buffer when it appears,
because the HDR source is the one with the picture in it. The tightened predicate refused the upgrade
too, so the run silently fell back to the washed-out early source and its "good" result was a worse
picture than before.

**The rule: tighten at the decision, not in the shared predicate.** The predicate answers *"could this be
the class of thing I want?"* — keep it geometric and broad. The **caller** answers *"may this be a first
source, or an upgrade to one I hold?"*, and that is where the format rule belongs. A recognizer used by
two decisions with different requirements must not carry either one's requirement inside it.

Two smaller findings from the same work, both cheap to inherit:

- **An allocation that arrives when nothing is waiting for it is lost for the rest of the process.** In
  that engine a target allocates on its **first use** and never again, so a latch that is not armed at
  that moment never gets another chance. If your capture depends on catching an allocation, arm it
  **before** the action that triggers the first use, not after.
- **A pooled resource can leave you holding a frozen buffer with no error anywhere.** When the engine
  pools by size, switching to a different size means your latched buffer is simply no longer written —
  the last frame it received stays on screen, indefinitely, looking like a hang in your own code.
  `[hypothesis]` on the pooling mechanism (2026-09-05); the symptom is measured. Detect it by comparing
  the size you are rendering for against the size you latched, and **warn rather than auto-clear**,
  because the same-size case recovers by itself.

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

## ⭐⭐ The camera you want may be a shipped rule you can enable from DATA — no code patch

`[verified-live 2026-09-07, n=1, observed at the controls]` Generalised out of
[`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr).

The section above says the setting you want to change may be data rather than code. Here is the same
idea at its most valuable, because the target is the thing a flat-to-VR conversion needs most.

On one 2008 title, the camera system is rule-driven: the shipped archive contains named camera rules,
each with a **list of state conditions** that must hold for it to apply — including developer rules
left in the retail data. **Rewriting one rule's condition list so that every condition is the
always-true state took over the live camera in normal gameplay, with no code patch at all.** The
edit was to the shipped archive; the only missing piece had been a repacker.

**Why this deserves its own entry rather than a line in the data-not-code section:** a camera
takeover is normally the expensive part of one of these projects — find the view matrix, find who
writes it, hook it, fight the engine for it every frame. A data-driven camera selector can hand you
the same thing for the cost of understanding one table. **Before designing a camera hook, look for a
camera *selector* and find out what decides which rule wins.**

Three things to check, in the order that costs least:

1. **Does the retail data still contain developer camera rules?** Names are the tell — anything of the
   `Debug`, `Free`, `Ghost`, `Fly` or `FirstPerson` family. Retail archives frequently keep them, in
   the same way retail builds
   [keep their assertions](#a-retail-build-that-shipped-its-assertions-names-its-own-globals).
2. **What gates them?** If it is a list of state conditions, the cheapest edit is to make the
   conditions trivially true rather than to raise a priority or add a new rule. In the worked case the
   priority half of the planned mod turned out to be **unnecessary** — the rule was already winning
   once its conditions passed.
3. **Which rule actually won?** Worth asking explicitly, because the answer was not obvious here: the
   observed behaviour was a *ghost cam* while the rule that had been patched was named for
   *first person* `[hypothesis]`. **A successful takeover does not prove you took over the thing you
   edited.**

**What you get is very unlikely to be what you want, and that is fine.** Here the result is a **free,
detached camera** — it flies into the sky and through walls, the movement keys still drive the
player, and normal play is impossible on that build. That is a *foundation*, not a feature: the
remaining work is locking it to the player's head, which is a data question about the same table
rather than a code question about the renderer.

### ⚠️ And the observation trap that came with it: a still frame cannot tell a locked camera from a free one

**Two readings were recorded wrongly from screenshots on the day this landed**, in opposite directions:

- *"The edit did nothing"* — taken **standing still**, which is the one state in which a free camera
  sits in a plausible third-person position and looks exactly like the shipped one.
- *"It produced a character-less camera"* — taken after the observer had **flown the camera away**,
  which looks like a bug and is just the feature working.

**The discriminating action is to MOVE the camera and watch whether the subject stays in frame.** A
still frame carries no information about the coupling between camera and subject, and coupling is the
entire question. This generalises past cameras: **when the property you are testing is a
*relationship* between two things, no single observation of either one can measure it** — you have to
move one and watch the other. Design the test as an action, not a screenshot.

See also
[check whether the game shipped a photo mode before building a detached camera](#check-whether-the-game-shipped-a-photo-mode-before-building-a-detached-camera),
which is the same instinct applied to a different shipped affordance — and note the counter-case
recorded there, where the bindings existed and dispatched nowhere. **A rule present in data is a
lead; a rule observed to change the picture is a finding.**

## Configure injected code from a file it reads itself, not from environment variables

`[verified-live 2026-09-04, n=3 launches]` A proxy took its knobs — including the one that enabled the
experiment being run — from environment variables. Three consecutive launches ran the experiment as an
**identity transform**, silently, and were written up as three uninformative results before the cause
was found: the launcher script that set the variables was not the thing that started the process.
**The file-based replacement is confirmed working on a storefront launch** `[verified-live 2026-09-04]`
— the same knobs armed correctly through the client that had been swallowing them, and the effect they
enable was visible on screen in that run.

**The mechanism is ordinary and easy to forget.** A child process inherits the environment of *its
parent*, unless the parent supplies a different block
([`CreateProcess`](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessw)
`lpEnvironment`). When a game is started **through a storefront client**, the parent is that client —
not your shell, not your script, not the terminal you exported the variable in. Nothing in the chain
reports a problem: your variable simply is not there, your code takes its default, and the default is
usually "do nothing". *(That a storefront client does not forward a user shell's environment is our own
observation and standard process behaviour, not something the vendor documents; treat it as the
expected case rather than a quirk.)*

**It is a silent no-op with an extra step**, and worse than most, because the missing configuration
also disables the very instrumentation that would have reported it.

**The rule: anything you must be able to set is read by your own code, from a file your own code
locates.** That project now reads an ini beside the executable, then a per-user path, with the
environment kept only as an override for the rare case where it genuinely is inherited. Three details
make it work:

- **Log where every value came from** — file path, environment, or built-in default — on the line where
  you log the value. Then a mis-set knob is visible in the first ten lines of the log rather than after
  three launches.
- **Put the file where the process will be, not where you are.** Beside the executable is the reliable
  location; the working directory of a storefront-launched game is not yours to predict.
- **Parse it with tests, not with confidence.** An ini parser is fifty lines and every one of them can
  swallow a value in silence. That project's is
  `[verified-numerically 2026-09-04, n=14 checks]` against a table of inputs, which is cheap insurance
  for something every future run depends on.

The wider principle this belongs to: **a launcher script is not a mechanism.** Anything that has to be
true at run time should be established by code inside the process, at the moment it is needed, and
recorded in the log — not arranged outside and assumed to have arrived. Generalised from
[`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr), 2026-09-04.

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

### The object you are writing to may not be the one on screen — read the flag back

`[verified-live 2026-08-19]` · `[measured 2026-09-06]` Generalised out of
[`arcade-controls-re2-vr`](https://github.com/TefMeister/arcade-controls-re2-vr) and
[`visceral-re2-vr`](https://github.com/TefMeister/visceral-re2-vr).

The section above is about writing to the wrong *value*. This is about writing to the right value on a
**dead object**, which produces the identical symptom — the feature does not respond — with none of the
same causes.

When an engine rebuilds the player (a save load, a death-and-continue, a level transition), the old
component objects frequently remain **allocated and writable**. A cached pointer taken before the rebuild
still accepts writes; the setter returns success; nothing throws. The meshes on screen are different
objects entirely. One project spent real time on a head-hiding flag that "did not work" and was in fact
being set perfectly, on a head that no longer existed.

**The guard is one line: read the flag back after clearing it.** If it reads the value you wrote and the
thing is still visible, your handle is stale — that is a *positive* discriminator, not an inference from
absence. Re-resolve on every event that could rebuild the actor, and treat a cached component pointer as
valid only for the frame you found it in.

**A second, subtler version of the same hazard is intra-frame.** In that engine a joint's cached
`WorldMatrix` is **stale if it is read in the same frame as a write to that joint** — one project read it
fresh at one update stage and stale at another, and the alternating pose was a hand-teleport bug that
looked like bad maths. If you write skeletal transforms, establish which stage the cache is coherent at
before trusting a read.

**And a mod that hooks a pipeline its own commits call needs a re-entrancy token.** The same project
guards its ammunition writes with a flag so its own commits are not intercepted by its own hook — and the
flag is **not refcounted**, so a nested commit clears it early and the outer one runs unguarded. In C++
this wants RAII with a depth count, not a boolean. The bug class is easy to miss because it only appears
when two of your own operations overlap, which is exactly the case a single-feature test never produces.

## ⭐ When every setter is a dead end, own the GETTER the solver reads

`[verified-live 2026-09-05, n=1 launch per weapon, 2 weapons]` The single most useful method result
of the 2026-09-05 sweep, and it generalises well beyond the engine it came from: **the way into a
system that refuses to be driven from the outside is often the value it reads from the inside.**

**The shape of the wall.** A game places a character's support hand on a weapon. Every *setter* the
engine's IK controller exposed was a dead end, in four different ways: one accepted a target every
frame and moved nothing (the game rewrote it from its own source before the solve); one solver kind
could not be enabled at all, by either the direct-ABI route or the reflective invoke route, with the
enabled flag reading back zero on every frame after; one target setter threw an internal exception on
every index; and two candidate solver objects were simply null at every read. Four negatives, all
correctly recorded, none of them progress.

**The move that worked was the opposite direction.** Instead of pushing a value into the solver,
find the **read path the solver consumes and override what it returns.** Concretely, a getter
returning the anchor's world matrix was post-hooked, and the returned translation edited. The wrist
went where the edited value said.

### The four steps, in the order that keeps each one cheap

1. **Locate the getter chain statically first.** Decompiling the "update constraint" routine showed
   it reading a *different* field than assumed — the weapon-to-right-hand attach, not the support
   hand — while the "get target" getter did nothing but return a joint's world matrix. That reframed
   the joint from a *follower* to an **anchor**, and named the thing to hook, before any launch.
   `[inferred-static 2026-09-05]`
2. **Prove the read path live with call counts, before building anything on it.** Post-hook the
   candidate getters and count calls per second next to the frame counter. Here both ran at exactly
   one call per frame (~345/s at ~350 fps) and the two counts were always **equal** — which, with
   step 3, also showed that one getter calls the other. **Zero calls would have been the whole
   answer**: it means the consumer reads natively and the hook must move down a level. This is the
   one question raw event counting answers well — see
   [counting events is not measuring content](#counting-events-is-not-measuring-content) for the
   many it does not.
3. **Override the return value and measure the effect end to end.** Add a known offset — 10 cm —
   and measure the *consumer*, not the hook: the skeleton's wrist joint moved 10 cm, joint-to-joint,
   read through a path with no hook in it. Shifting **both** getters in the chain moved it 20 cm,
   which is a two-line proof that they nest. Switching the override off returned the distance to
   0.000, including under the game's own aim state.
4. **Ask whether the consumer blends or snaps**, with a trace across the toggle edge. Here it
   snapped inside one 100 ms sample — so any "smooth, not jarring" requirement is the mod's to
   provide, by blending the value it returns. That is trivial when the hook already produces a
   fresh value every frame, and it is much less trivial if you discover it after shipping.

### 🚨 The trap that cost two launches: the value is read in a *pre-update* pose, and the consumer applies your DELTA

`[verified-numerically 2026-09-05, n=6 samples, fitted to ≤2 mm]` This is the part that does not
appear until the override is already working, and it is the reason a hook that measurably moves the
right thing can still land in the wrong place.

The getter is called at **one specific point in the frame**, and the quantity it returns is that
quantity *as it is at that point* — not as it will be when the frame is finished. Measured here, the
gap was **0.18–0.20 m and about 48°** between the value the game's own per-frame call returned and
the same joint's final pose (18° on a second weapon — so the gap is per-weapon and per-stance, not a
constant you can hard-code). A read taken from a late, pre-render hook returned the final pose; the
getter did not.

**And the consumer carried the offset, not the value.** The solver applied the *difference* between
what the getter returned and what it would naturally have returned, transported into final space —
so an absolute target expressed in final space missed by exactly that gap, every frame, while still
producing a convincing 10-cm response to a 10-cm test edit. **A relative test cannot distinguish a
relative consumer from an absolute one**, which is why step 3 above looks like a complete success and
is not.

The general rule to carry: **before writing an absolute value into a hooked getter, establish which
space and which moment the value lives in**, by reading the same quantity a second way — a different
hook point, or the engine's own accessor for the same object — and comparing. If the two disagree,
you have both the mapping between them and the knowledge that you need one. Here that mapping was a
single matrix recomputed live every frame from the un-hooked value and the object's own accessor, so
the fix was *blend in final space, then map back* — not a constant, and not a guess.

### Two smaller traps from the same work, both cheap to inherit

- **A post-hook may hand you a pointer to the register holding the hidden return-buffer pointer, not
  to the struct.** Dereference once, then check the nullable's `HasValue` byte before touching the
  payload. Silent nonsense otherwise.
- **Your own reads of the getter go through your own hook.** Useful as a self-check that the edit
  lands; actively misleading if you do not subtract them from the call count (here about 1/s of
  plugin traffic against the game's ~345/s), and it explains a genuinely confusing earlier result —
  the getter had been recorded as *"returns no value"* on one weapon, when what had actually been
  sampled was the plugin's own dump-time call, not the engine's per-frame one.

Generalised from [`visceral-re2-vr`](https://github.com/TefMeister/visceral-re2-vr) (RE Engine via a
REFramework C++ plugin), whose §8c carries the engine-specific detail; the method needs none of it.
See also the [RE Engine page](../engines/re-engine.md).

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

### …and it must FREE the real DLL on detach, or a reload walks straight past it

A second, quieter way to lose the same foothold, and the failure looks nothing like a bug in your code
`[reported, first-party 2026-09-04]`.

**The loader rule that causes it** is in Microsoft's own `LoadLibrary` remarks: *"When no path is
specified, the function searches for loaded modules whose base name matches the base name of the
module to be loaded. If the name matches, the load succeeds. Otherwise, the function searches for the
file."*
([LoadLibraryA](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibrarya)
— the remark is on that page specifically, not on `LoadLibraryEx`.) A bare module name resolves to
**whatever is already resident under that name**, and the search path is never consulted.

**How that removes your proxy.** Your game-folder `d3d9.dll` loads the real one from the system
directory by full path — correct, and the standard pattern. If the game then `FreeLibrary`s *your*
proxy — a capability probe at startup, a renderer restart, an options change — and your `DllMain`
never released the system copy on `DLL_PROCESS_DETACH`, the system copy stays resident under the base
name `d3d9.dll`. The game's next `LoadLibrary("d3d9.dll")` matches it by name and succeeds
immediately. **The application directory is never searched, your proxy is never loaded again, and the
game runs perfectly without you for the rest of the process.**

**The log signature, which is the useful part.** Per launch, the proxy log contains a load, one or two
export calls, and an unload within about a hundred milliseconds — and then nothing, while the game
visibly reaches gameplay. That reads as "the game crashed my mod" or "this game must use a different
graphics API". It means neither. It means **you were reloaded past.**

**Prior art confirms both the failure and the fix.** ReShade carried this bug against one specific
game until commit
[`74347b91d`](https://github.com/crosire/reshade/commit/74347b91d7729a6da93040298c6587bb3b786da4)
(2019-12-19, shipped in 4.5.2), whose title is simply *"Fix hooking in Alan Wake"*; the reason is
written as a comment in the diff itself — freeing the reference to the module loaded for export hooks
*"is necessary for Alan Wake to work"*. The same game, the same one line, seven years apart.

**⭐ Confirmed on a real title, 2026-09-04.** The project this came from went back to its own logs and
found the game loads `d3d9.dll`, calls the creation export once, unloads it about **6 ms** later, and
then loads `"d3d9.dll"` a **second** time for the device it actually renders with `[measured
2026-09-04, n=3 launches]`. Its proxy held the system module the whole time, so that second load
matched the resident copy by base name and the game folder was never searched again. **That is the
entire explanation for a proxy that "only ever sees one short-lived call"** — a symptom the project had
carried as an unexplained disagreement between its recorded status and its surviving evidence.

**There are two fixes, and the second one is better.**

1. **Release the real module in `DLL_PROCESS_DETACH`** — but with a guard, and knowing the trade.
   **Microsoft documents both halves of this.** The entry point *"must not call the **FreeLibrary**
   function (or a function that calls **FreeLibrary**) during process termination, because this can
   result in a DLL being used after the system has executed its termination code"* — and the way to
   tell the two cases apart is the third parameter: on `DLL_PROCESS_DETACH` it is **NULL if
   `FreeLibrary` has been called** and **non-NULL if the process is terminating**
   ([`DllMain` remarks](https://learn.microsoft.com/en-us/windows/win32/dlls/dllmain), and the same
   guidance in [dynamic-link library best practices](https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-best-practices)).
   So: **free the real module only when that parameter is NULL.** Even then you are calling
   `FreeLibrary` from inside `DllMain`, which the same guidance discourages in general; it is done here
   because the unload is the only moment the reference can be released, and because ReShade ships the
   same call for the same reason. **If a launch later hangs at exit or on the second load, this is the
   first suspect.**
2. **Or never load a module under the same base name at all.** A proxy that loads a *renamed* copy of
   the original — `Something_Original.dll` beside the game rather than the system `Something.dll` —
   cannot be bypassed this way, because a later `LoadLibrary("Something.dll")` has no resident module
   of that base name to match. The rename pattern is usually adopted for a different reason (games
   whose real DLL is a game-folder file, not a system one), and this immunity comes free with it.

**⭐ The fix is confirmed live** `[verified-live 2026-09-04, n=1 launch]`. On the title above, the same
log now shows — in **one process** — the probe load, the creation call, an unload that records itself as
an explicit `FreeLibrary` rather than a process teardown, **a second "proxy loaded" block with the same
process id**, and then no further unload for the rest of the session. Both the title screen and the menu
rendered through it. **That second block in the same PID is the acceptance test**; it is unambiguous, it
costs nothing to log, and it is what "the game found us again" looks like. For that project this was the
central unblock: for weeks its proxy had only ever seen a throwaway probe device, and it now owns the
device the game actually renders with. The loader-lock caveat above **did not bite on that launch** —
one data point, not a clearance.

**Audit every proxy you own, because a leak is a *latent* failure**: it works perfectly until it meets a
game that probes-and-reloads, and then costs a session to diagnose from scratch. That audit is a grep
for `FreeLibrary` in each proxy's `DllMain`, and it is worth actually running rather than assuming.
Doing it across one estate `[inferred-static 2026-09-04, n=10 proxies read]` found that **of the eight
proxies that load the real system module by path, exactly one released it**; two apparent passes turned
out to be the word `FreeLibrary` appearing in a *comment*, which is a reminder to grep for the call and
then read the line. The one proxy that was structurally safe was safe by fix 2, not fix 1 — it loads a
renamed original.

Generalised from a `/gr` research hand-off on
[`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr), 2026-09-04.

#### 🚨 …and the reload the fix enables has a race of its own: the slot you chain into may not be the engine's

`[verified-live 2026-09-05, n=1 crash, n=3 clean launches]` Found the day after the fix above, on the
same title, and it is the fix's own consequence rather than a separate problem — **so read this
section as the second half of the one above, not as an unrelated caution.**

A proxy that is loaded, unloaded and loaded again now installs its vtable patch **twice**, and the
second install reads the current slot contents as "the real function" to chain into. That is correct
only if nothing else patched the slot in between. Here it was not: at the first unload the slot held a
**foreign pointer** — a third-party overlay is the likely owner — so the unhook correctly stood down
rather than clobbering someone else's work, and then the second load captured *that* pointer as its
"real" one. The two patches chained into each other and the hooked function recursed **1,669 times in
one millisecond**.

The distinguishing evidence that it is a race and not a design error: the first proxy block lived
**700 ms** on the launch that crashed and **16 ms** on the three that did not. Same binary, same
machine, four launches.

**The rules that fall out of it, none of which cost anything:**

- **Record the pointer you replaced, and on re-install compare the slot against what you left there.**
  If it differs, someone else owns the slot now — do not capture their pointer as the original.
- **Never chain into a pointer you did not verify came from the module you expect.** A slot value that
  resolves outside the graphics runtime's own module range is a foreign patch, and that check is one
  `VirtualQuery` or one module-range comparison.
- **Guard against self-recursion at the call site anyway**, with a per-thread depth counter that bails
  above one. It is three lines, it converts an unrecoverable stack overflow into a logged anomaly, and
  it is worth having even when you believe the chain is clean.
- **The correct behaviour on unhook — stand down when the slot is not yours — is what creates the
  window.** It is still the correct behaviour. The bug is not in the unhook; it is in the *next*
  install trusting a slot that the unhook already knew was compromised. Those two decisions are
  usually in different functions written weeks apart, which is exactly why this shape survives review.

Generalised from [`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr), 2026-09-05.

## A vtable patch is a LIFETIME commitment — restore it before anything can unload you

`[inferred-static 2026-09-04, n=1 title]` The companion hazard to the state-block rewrite above, and
between them they cover most of the ways an in-place vtable patch fails without saying so.

**The mechanism.** You write a pointer to your own function into slot *n* of a COM interface's method
table. That pointer is an address **inside your DLL**. If your DLL is then unloaded while the patch is
still installed, the slot holds an address in **unmapped memory**, and the next call through it is an
access violation with a stack that points at nothing recognisable. **A D3D9 vtable is shared per
interface class, not per object**, so the patch outlives every object you saw it through — releasing
the interface does not unwind it, and neither does the game creating a fresh one.

**The worked case is instructive because the wrong conclusion survived for ten days.** A project
hooked `IDirect3D9::CreateDevice` in place, the game crashed, and the hook was marked *confirmed
broken — something about how this patch applies to this game's vtable, not yet understood*. Every
part of that is wrong except the crash. The game **unloads the proxy about 6 ms after the creation
call** (see the section above), nothing ever restored slot 16, and the next call jumped into the hole
where the DLL had been. Mechanical, reproducible and not specific to the game at all — and the
"cause unknown" label is what stopped anyone looking, since a mystery invites avoidance rather than
five minutes of reading.

**What to do, in order:**

- **Write the unhook when you write the hook**, not when you need it. Restore the runtime's own
  original pointer, and **refuse to touch the slot if it no longer holds yours** — a later hook may
  own it now, and stamping your "original" over that breaks somebody else's mod.
- **Unhook BEFORE releasing the real module.** The vtable usually lives in the system module's own
  data; once you have freed your reference to it, writing there is a race at best.
- **Assume you can be unloaded at any time.** A capability probe, a renderer restart or an options
  change can unload a graphics proxy mid-session — this is the same reload behaviour that bypasses a
  proxy which never frees the real DLL, seen from the other side.
- **Beware the dead-code trap when the hook is behind a compile-time switch.** With the hook disabled,
  an optimiser will strip the unhook path too, so the shipped binary contains neither. That is
  correct, but it means the unhook is **unverified** until something builds it in. The project above
  proved theirs compiles by building a scratch copy with the hook enabled, confirming its log strings
  appear, and discarding that build — a cheap habit worth copying for any code that only exists in a
  configuration you do not ship.

**And the claim-hygiene half:** a verdict of "confirmed broken, cause unknown" on a standard technique
should read as an open question, not a closed one. This library's own
[Remedy engine page](../engines/remedy-alan-wake.md) carried the mystery version for ten days, and the
correction cost one careful read of a lifetime, not an experiment. Generalised from
[`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr), 2026-09-04.

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

### The diagnostic that is gated on the failure it was written to explain

A related and very cheap mistake, from a different project the same week. A proxy bucketed the draws
its patcher had missed — sizes, usage flags, shader hashes, everything needed to answer *do the
missed draws carry world geometry?* — and printed the table **only while the patched count was
zero**. Once patching started working, which was the whole point of the session before it, the table
stopped printing. The instrumentation existed, the data existed, and the question sat open for a day
behind one `if` `[compile-verified 2026-09-04]`.

**When you gate a diagnostic on a failure condition, you delete it at the moment the thing half
works** — which is exactly the state in which its answer is most interesting. Print periodically and
at shutdown instead, and let volume be managed by the interval rather than by a predicate on
success. Generalised from [`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr).

**A second project met the same shape the same week, and its escape is worth stealing.** A first-ever
proxy launch was meant to answer *does our per-eye edit reach the screen?* and came back
**inconclusive** for three instrumentation reasons at once: the one-shot statistics line fired before
the hotkey that enabled the feature, the write counter was gated behind the enabled flag, and the
hotkey handler sampled the value of interest **at the toggle instant** — so its `not seen yet` was
produced by construction rather than measured. The frame diff that was supposed to back it up showed
only scene animation. **A result like that is not a negative**, and recording it as one is how a
working lever gets written off.

**The escape needed no rebuild: toggle twice.** On → let enabled frames actually run → off → on, then
read the **second** enable line, which now samples after the feature has had frames to work in. That
retest turned the same session's inconclusive into a confirmed result. **Whenever a diagnostic
samples at the moment you flip something, flipping twice is the free fix** — and it is worth trying
before rebuilding the instrumentation, because it costs one launch you were having anyway.
Generalised from [`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr),
2026-09-04.

### An instrument that tests one convention can only ever report "neither convention"

`[verified-live 2026-09-05, n=1 launch]` The third shape in this family, and the most quietly
expensive, because it produces a *stable, plausible, repeatable* answer that is wrong by
construction.

A dump was built to answer one question: is the engine's view-to-clip matrix left- or right-handed?
It ran, it logged, and for two launches it reported that the uploaded matrices matched **neither**
handedness. Two independent defects, either of which alone would have been enough:

- **It searched at the wrong granularity.** The engine uploads shader constants in **whole
  128-register blocks**, thousands of times a second, and the dump looked for an upload whose start
  register equalled the register of interest. That condition never fires. Every block was attributed
  to register zero, and what got printed was whatever happened to sit at the head of the block — a
  flat 2D matrix, at around 500,000 uploads a second, which is a very convincing-looking wrong
  answer.
- **It tested the storage convention the project had already ruled out.** It checked for the
  telltale ±1 at the array index implied by one matrix convention. The project's own notes had
  settled two days earlier that this engine uses the other one, where that entry lives at a
  different index. So the test could return "neither" and nothing else, **forever**, no matter what
  the engine did.

**The two habits that would have caught it, both cheaper than the launches it cost:**

- **Make the instrument print the population it is searching, not just its verdict.** A histogram of
  every `(start, count)` upload range — which is what the replacement does — states in one line that
  the traffic is 128-register blocks, and the granularity bug is then unmissable. A search that
  reports only hits cannot distinguish *"the thing is not there"* from *"I was not looking at the
  shape of thing that exists"*.
- **Scan for every convention you have not disproved, not for the one you expect.** The replacement
  slides a four-register window across each block and tests **both** signatures. That is a few more
  lines and it removes the entire failure mode — and when it ran, the answer arrived on the first
  launch, from three separate projections at once, agreeing with each other.

This is the same family as the two sub-sections above and worth naming separately because the
symptom differs: a gated diagnostic goes *silent*, and a convention-blind one stays *loud and
negative*. A confident, repeated negative from your own instrument deserves the same suspicion as a
crash. Generalised from [`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr), 2026-09-05.

### A log line that co-occurs with a failure is not an explanation of it

`[disproved 2026-09-05]` The cheapest mistake in this whole section, and it cost a design decision
rather than a launch.

A diagnostic block printed three counters together: a pairing table reporting **zero** successful
writes, the same table reporting **2.8 million** overflows, and — in the same block — a line reading
*"thread-ring pool exhausted (8 distinct threads seen)"*. The obvious reading was that a
thread-indexed pool sized for eight threads had run out, and the obvious fix was to make it bigger.

**The line belonged to a different subsystem entirely** — an unrelated pool of draw-time scratch
buffers — and its own message text went on to say that the patch stays correct through that
condition. Sizing the pool would have changed nothing at all, and the real cause (a table keyed on
the wrong identity) would have survived the fix and been that much harder to see afterwards.

**The guard is one question, asked before the reasoning starts: which code emits this line?** In a
proxy or a hook that carries several independent subsystems, one diagnostic block is a *place*, not
a *story*, and adjacency in a log is not causation. Two small habits make the question answerable in
seconds — **prefix every log line with its subsystem tag**, and **read the emitting site, not the
message**, because a message written for one context reads as an explanation in another. Generalised
from [`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr), 2026-09-05.

### A hard-edged mask makes phase correlation lie, confidently

`[verified-numerically 2026-09-05]` A fourth entry in this family, and the first where the instrument
is a **measurement** rather than a diagnostic — which is why it was believed for a whole day.

Measuring how far a picture moved between two captures is a solved problem: high-pass both images and
phase-correlate. The complication in a VR mod is that the interesting picture is usually a *region* —
the inside of a scope, a mirror, a portal — so the natural move is to mask everything else off with a
binary annulus and correlate what remains.

**Do not.** A hard-edged mask has enormous energy at its own rim, and after high-passing, that rim is
the strongest feature in **both** images and sits in exactly the same place in both. The correlator
locks onto the mask and returns precisely `(0, 0)` with a peak-to-rms ratio around 500 — a confident,
well-conditioned, completely fabricated answer. Every *"the picture did not move"* result that project
recorded in one day came from this. Use a **smooth raised-cosine taper in radius** instead.

**The control that catches it — and nothing else did — is a known offset.** Roll or shift the baseline
image by an amount you chose, and require the pipeline to recover that amount before you believe
anything it says about real data. The hard mask recovered `(0, 0)`; the tapered mask recovered
`(-23, +17)` exactly, and pinned the sign convention as a free bonus. This is
[validate the instrument before you trust either result](#3-validate-the-instrument-before-you-trust-either)
in its cheapest possible form, and an image pipeline should carry it permanently.

**Two more ways to build the mask wrong**, from the same work: a mask derived from "high variance"
regions locks onto the **animated world outside** the region of interest, which does not move with the
thing you are steering, and again returns a confident `(0, 0)`. And whole-frame mean-absolute-difference
is worse than useless — a pair of frames that differ unmistakably to the eye scored 14.6 against a
same-state noise floor of 10.0.

**The escape hatch worth remembering:** a large, unmistakable action, looked at. A 40° step settled in
one move what two correlators could not settle in a day.

### The noise floor is the idle animation, and it can exceed the effect

`[measured 2026-09-05]` The companion trap to the one above, and the reason a fitted slope from that
project had to be withdrawn.

Two captures of the *same commanded state*, seconds apart, differed by up to 26 pixels — because the
player character idles, and the weapon sways with them. Nothing was commanded; nothing was wrong. That
is the floor every measurement in that scene sits on.

The consequence is arithmetic. Across a 2.5° sweep the quantity being measured moved **less than the
idle**, so the fitted slopes (`dx` −3.5, `dy` +4.9 px/deg) carried residual RMS of 7.0 and 6.0 px —
larger than the total change across the entire sweep. A slope like that is not a weak measurement, it is
not a measurement. It was published as a lower bound, and withdrawn a day later.

**The rule: take same-state repeats first, and quote the floor beside every number derived from that
scene.** Three or four captures with nothing commanded cost seconds and tell you the smallest effect the
scene can express. If the effect you want is smaller, the answer is a bigger step, landmark tracking, or
suppressing the idle — not a better fit. See also
[prove the test can fail](#prove-the-test-can-fail-mutation-check-a-numerical-verification-before-trusting-it).

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

- **`D3DPOOL_MANAGED` does not exist on a D3D9Ex device** — any `CreateTexture` /
  `CreateVertexBuffer` / `CreateIndexBuffer` asking for it **fails**. This library first recorded
  that as *"the difference between a one-line proxy change and a resource-remapping project, not
  answerable statically"*. **Corrected 2026-09-03: it is a solved, bounded proxy problem, and it is
  not a gate.** `[reported 2026-09-03]` The established rewrite, in the proxy's own `Create*`
  wrappers, is **`D3DPOOL_MANAGED → D3DPOOL_DEFAULT + D3DUSAGE_DYNAMIC`**, and it is safe for a
  reason rather than by luck: Microsoft's own `D3DPOOL` reference says MANAGED exists so the runtime
  can restore resources after **device loss**, and its "Lost Devices" page says a 9Ex device **never
  returns `D3DERR_DEVICELOST`** — so on 9Ex the pool has nothing left to do, and translating it away
  is the migration the design implies. The `DYNAMIC` half is load-bearing: DEFAULT textures cannot be
  locked unless dynamic, whereas MANAGED ones always can, so plain DEFAULT would make every
  allocation succeed and then fail at the first `Lock()`. DEFAULT × DYNAMIC is a legal pairing and
  MANAGED × DYNAMIC is not, so the rewrite can never collide with a usage flag the game already set.
  **Prior art:** `elishacloud/dxwrapper`'s `D3d9to9Ex` option does exactly this, following Special
  K's strategy, and its maintainer reports **7 of 8 tested games working**. The same source names what
  actually breaks on 9Ex, which replaces one vague unknown with four observable ones: **paletted
  textures** are unsupported; **16-bit textures** only work in system memory; **D3DX functions remain
  problematic** (relevant to any 2008–2011 title with a D3DX dependency chain); and **some titles
  fail outright at device creation**. All four are cheap to observe on the first Ex launch. An
  instrumented MANAGED count is still worth having — it sizes how much `Lock()` traffic gets
  re-pointed — but it is a measurement to ride along on a launch that is happening anyway, not a
  prerequisite in front of static work. Sources:
  [D3DPOOL](https://learn.microsoft.com/en-us/windows/win32/direct3d9/d3dpool),
  [Lost Devices (Direct3D 9)](https://learn.microsoft.com/en-us/windows/win32/direct3d9/lost-devices),
  [dxwrapper discussion #105](https://github.com/elishacloud/dxwrapper/discussions/105).
  Found by a `/gr` pass for `enslaved-vr`, whose dossier adopted it the same day and un-gated its
  D3D9Ex route.
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

**⚠️ The sibling failure: a test that passes because it tested nothing.** Mutation-checking asks *can
this test fail?* There is a cheaper and more common failure it does not always catch — a test whose
assertions never execute, or execute on values that are all zero. One project's first parity test
passed cleanly while asserting nothing at all: its sample matrix was not classified as a perspective
matrix, so the code path under test skipped every comparison and each assertion reduced to `0 == 0`
`[verified-numerically 2026-09-04]`.

**A test satisfiable by "nothing happened" is worse than no test**, because it is counted as evidence.
The fix is one assertion, placed first: **assert non-vacuity** — that the classification succeeded,
that the loop ran a non-zero number of times, that the value under test is not the neutral one. Then
the mutation check has something to break. This is the same shape as the
[silent no-op](#silent-no-ops-verification-that-cannot-see-the-failure) one level up, in the harness
rather than in the mod.

Generalised from [`unreal-gold-vr/modding-notes/`](https://github.com/TefMeister/unreal-gold-vr/tree/main/modding-notes)
(`2026-09-02-m2-stereo-proof-built-verified-deployed.md`); the test output and mutation record are in
that repo's `dev-archive/recon/`.

## Test a runtime-compiled shader without the game: one file, two compilers

`[compile-verified 2026-09-03]` for the build, `[verified-numerically 2026-09-03]` for the harness;
`n=1` project. The section above proves stereo maths off-line by compiling the DLL's own header into a
console test. It does not cover the other place maths hides in a VR plugin: **HLSL handed to
`D3DCompile` at load time as a string.** Two consequences for a no-launch lane — a typo surfaces only
at the next launch, and the curve inside the shader can be tested only by transcribing it into a
harness, at which point the harness tests the transcription and not the shipped bytes. A stereo
harness in this estate caught exactly that distinction the hard way.

The route that closes both gaps, worked once on a compositor plugin's tone curve:

1. **Write the maths in the C/HLSL common subset** — scalar `float`, arithmetic, `?:`, `pow`, `exp`,
   `f`-suffixed literals; no vector types, no HLSL-only intrinsics such as `saturate` or `smoothstep`,
   no `double` literals. Keep it in its own file (`.inc`).
2. **Compile the same bytes twice.** `#include` the file into the plugin as C++ (a CPU-side inverse is
   a natural consumer); and have the build system read the file into a generated header as a raw
   string that the plugin prepends to its HLSL before `D3DCompile`. With CMake that is a
   `file(READ)` into a `configure_file` template, plus `CMAKE_CONFIGURE_DEPENDS` on the `.inc` so
   editing it re-runs the step.
3. **A numeric harness includes the `.inc` directly** and checks it against an *independently
   derived* reference plus the analytic properties the design relies on — here, a transcription of the
   published curve (maximum deviation 5e-8), the straight section exactly straight, value and slope
   continuous at the shoulder, and the inverse round-tripping. The harness tests the bytes the GPU
   runs.
4. **Run `fxc` over the assembled source**, so the shader is *compiled* without the game. A short
   script concatenates the `.inc` with the shader string extracted from the source file and compiles
   every entry point with the Windows Kits compiler. One gotcha for Git Bash users: MSYS rewrites
   `/T` into a drive path — use `-T` and `MSYS2_ARG_CONV_EXCL="*"`.

**The trick that does not work:** wrapping a raw-string delimiter in `#ifdef` so one file is both
code and string. Raw-string tokens are formed before conditional inclusion, so the skipped branch
still swallows the code. Do it in the build system.

A bonus habit from the same note: **published defaults are a fingerprint.** When a live engine exposes
tonemap parameters that match a named curve's published defaults to the digit, that identifies the
curve strongly enough to build against — and, as here, the identification can turn out to matter
(a "length" parameter that is a fraction of headroom, not an absolute). The RE Engine instance is on
[its family page](../engines/re-engine.md#the-games-tone-curve-is-a-published-one-and-its-parameters-are-readable).

Generalised from [`re-village-scope-vr/modding-notes/`](https://github.com/TefMeister/re-village-scope-vr/tree/main/modding-notes)
(`2026-09-03-tone-curve-gt-shoulder.md`); the harness and check script are in that project's
private staging tree, and the method is described here entirely in our own words.

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


**✅ Measured the next day — the gap is real, and the design changed because of it.** `[measured
2026-09-03, n=26,595 frames / 343 one-second intervals / 3,254,942 draws, one session]` The recon
above was run with the denominator, on the same UE2/D3D8 title: programmable-VS draws were present in
**338 of 343** intervals, a frame-weighted **8.72 %** of all draws, peaking at **51.4 %** of one
second's draws (mean 11.8 per frame, max 113.7), from only **six** distinct vertex shaders in the
whole session; the handle table did not overflow, so the figure does not understate. The
all-fixed-function seconds were menus and loads — **the busiest gameplay seconds had the highest
programmable share**, the opposite of the reassuring pattern. Decision: draw-twice stays the
backbone (~91 % of draws, no early-out to fight) **plus a vertex-shader-constants path** for the
rest; the live constant uploads were `c0 ×5` (confirming the static `+0x4205` inference), `c0 ×8`
and `c10 ×1`, so the per-eye matrix's destination is known even though *what* those draws are is
still `[hypothesis]`. **Do not carry the 8.72 % across engines** — the transferable part is the
question and the method, not the value; another engine could be 0 % or 60 %.

Two further measurements from the same run are worth copying into any draw-twice recon:

- **Split projection sets into perspective vs orthographic per frame.** Here orthographic
  outnumbered perspective **4.6 : 1** (12.9 vs 2.8 per frame) — most projection traffic is HUD and
  canvas, so a stereo path that rewrites *every* projection corrupts the interface far more often
  than it fixes the world.
- **Count distinct view matrices per frame.** Vanilla already issued **3–8** distinct views per frame,
  so "the view matrix" is not one thing and the player camera has to be **identified**, not assumed.

And one instrument trap: a bounded matrix-dump budget (`N` dumps per run) was **spent entirely on the
menu** — every dump carried the same pre-gameplay frame number, so no gameplay matrix was captured
while the per-interval counters were fine. Gate dump budgets on a frame threshold or a key, not on
"first N".

### ⭐ The cheapest coverage test is an ABSURD transform, and it is a picture

`[verified-live 2026-09-04, n=1 game]` Everything above measures coverage in counters. There is a
test that measures it with your eyes, needs no new instrumentation, and is unambiguous: **apply a
deliberately ridiculous transform — a 90° yaw is ideal — to every draw your patch reaches, and look at
what stays still.**

Whatever remains upright and un-rotated is exactly the geometry your patch does not cover, rendered in
place, at its real size, in its real proportion of the frame. On the worked case the opening scene came
back radically transformed with unrotated fragments through it and a detached, upright character head —
which settled in one launch a question a whole session of counter-reading had left open: **the missed
draws carry real world geometry.** They were not skinning, not effects and not decals, and the
residual was therefore not harmless.

**Why it beats the counter it replaces.** A per-draw patch reports "patched" and "skipped" totals, and
the skipped total is compatible with two completely different worlds — a large number of tiny
irrelevant draws, or a small number of draws carrying the level. Counts cannot separate those; a
picture does it instantly, and it also shows you *what kind* of thing is missing, which no counter
does.

**Three practical notes.** Use a rotation rather than a translation — a rotated world is
unmistakable where a shifted one can read as camera motion. Pick a scene with recognisable large
geometry rather than a corridor. And keep the absurd mode as a permanent, hotkeyed diagnostic rather
than a temporary hack, because it answers "did my coverage change?" after every extension of the patch,
which is the question you will ask most often.

Generalised from [`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr), 2026-09-04,
where it also confirmed that per-shader dynamic constant buffers — not just the shared pool — must be
covered before a stereo build is worth judging.

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

## Two-handed VR weapons: the second controller hides behind the first

`[reported 2026-09-05]` from this account's own headset time. **Corrected 2026-09-07** — an earlier
version of this section described one mod's offset as *"per weapon, configured in LTX"*. **That clause
is withdrawn**; see the method note at the end, which is the most transferable thing here.

Hold a rifle the way a person actually holds one and the support hand ends up **directly behind the
trigger hand along the headset's line of sight**. On an inside-out headset that is the worst case for
optical tracking: the rear controller is occluded by the front one, its pose degrades immediately, and
the weapon jitters or swings. Observed live in a RE Village session on 2026-09-05, where it cost a
retake. This is not a tuning problem — it is a geometric consequence of the natural pose, so **it will
appear in every project that ships a two-handed weapon**, and it is worth designing for before the
first headset test rather than after.

Meta's own tracking write-up names the condition without quantifying it: *"Scenarios that suffered the
worst are when the controllers are near the edge of field of view, too far, too close, or when there is
occlusion."* `[reported]` (read firsthand 2026-09-07). A targeted search found **no vendor-published
figure** for how long a controller's pose coasts on its IMU once occluded — neither Meta nor Valve
appears to publish one, and the community explanations that exist are not specifications. Treat "how
bad, and for how long" as unknown; design so the question does not arise.

### The published design families

Every shipped solution breaks the 1:1 mapping between physical controller and in-game hand — that
mapping is what forces the two controllers into line — but they break it in different places, and the
choice is really about **authoring cost**.

| family | mechanism | who ships it | what it costs |
| --- | --- | --- | --- |
| **Stop reading the rear hand for aim** | the front hand and body drive orientation; the rear hand stabilises only | **Onward** (*Virtual Gunstock*), **H3VR** (`use gun rig mode`) | *"a slight loss of fine control"*; and it **breaks weapons whose foregrip legitimately sets the angle** — H3VR's lever actions will not cycle with it on |
| **Offset the rear hand's IK target** | the secondary hand is *spread apart* in IK so the controllers do not cover each other | **STALKER Anomaly VR** (MarsyApp) | proprioceptive mismatch scaling with the offset |
| **A named second-hand attach transform authored on the item** | not numbers in a config — a handle transform that is part of the asset | **Blade & Sorcery** `[hypothesis]` — reached via a search summary only, not a fetched page | per-asset authoring, but it is *content*, not a config table |
| ~~a numeric per-weapon offset table~~ | — | **nobody publishes one** | the one studio publicly asked for it **declined** |

**⭐ The judgement to take away is the authoring-cost argument, because a developer made it in public.**
An H3VR player asked for per-weapon offsets so different rifles would align consistently on a physical
gun stock. The developer's answer `[reported 2026-09-07, verified firsthand]`:

> *"There's nothing I can do about this that wouldn't be incredibly time consuming, and require me to
> generate an extra entire set of manual poses."*

H3VR ships a **global** toggle instead — `use gun rig mode`, which makes every gun's forward direction
consistent and stops the foregrip grab from determining the facing angle — with the honest caveat that
lever actions depend on exactly that behaviour and are incompatible with it.

**That objection scales with weapon count, and that is the whole of it.** For a gun sandbox with
hundreds of firearms, a per-weapon pose set is a content programme and the global toggle wins. For a
game with a handful of weapons — which is most of what this account mods — the objection is weak and a
small per-weapon table is cheap. **Read the refusal as a cost argument, not as a design verdict.**

**⚠️ What remains genuinely unknown about the offset itself.** MarsyApp's published material says the
secondary hand is *spread apart* (`разводится`); it does **not** say *above*. This account's own
observation is that the left hand should be held **above** the right, stacking the controllers
vertically `[reported 2026-09-05, n=1 observer]`, and no screenshot or video confirming the real-world
hand geometry could be found. So the **direction** of the offset is a knob to find in the headset, not
a constant to copy. What is published on that mod points at a **user-calibrated runtime value** with an
in-headset calibration tab (its roadmap lists VR Tools calibration tabs and an in-game settings
section, and every documented console variable is global) rather than a shipped table `[hypothesis]`.

**The cheap test, before writing any IK:** hold the pose in the headset with the mod's existing
one-to-one hands and watch the rear hand. If it jitters, an offset is needed; how much, and in which
direction, is what one session with a slider settles.

### ⚠️ The method note, which is the reason this section was corrected

The withdrawn clause — *"per weapon, in the game's LTX config files"* — **never appeared in the body of
any page that was actually fetched.** It existed only in summarizer prose, and it survived a
verification pass **because the verification question named the string it was verifying**: the fetch
was asked to quote any sentence about "per-weapon LTX configuration of grip points", and it obligingly
produced one. That is precisely the failure documented at
[never name the string you are asking a fetcher to find](#-and-the-false-positive-never-name-the-string-you-are-asking-a-fetcher-to-find)
— committed, in this case, by the check that was supposed to catch it.

**Two rules follow, and they are worth more than the finding they cost:**

- **The don't-name-the-string rule binds VERIFICATION fetches too, not just discovery.** A confirming
  question is the *easiest* one to answer wrongly, because it supplies the shape of the answer. Verify
  with an open prompt — *"summarise this thread and quote the developer's replies verbatim"* — which is
  how the H3VR quote above was obtained.
- **⭐ A claim that appears only in summarizer prose, and never in the body of a fetched page, is not
  `[reported]`.** It has no source yet. That is a mechanical test anyone can apply, and it would have
  caught this before publication.

**One deliberate limit, recorded so nobody re-spends it:** that mod is closed-source, ships through its
own launcher, and its README lives inside the archive — the likeliest home of any real key names, and
out of reach, because this library does not download other people's mods to study them. And
`h3vr.fandom.com` returns **HTTP 402** to direct fetches (three URL forms tried); its content reached
us through search snippets and the Steam thread instead. Effectively checked, not directly fetched.

Sources, all read online, nothing downloaded: **MarsyApp** — Anomaly VR's own development thread and
Boosty posts (Russian): <https://ap-pro.ru/forums/topic/14575-anomaly-vr/> ·
<https://boosty.to/anomaly_vr>. **UploadVR** on Onward's inside-out tracking update:
<https://www.uploadvr.com/onward-inside-out-tracking-update/>. **Anton Hand / RUST LTD** and
**[RUST]Grumplestiltskin** for the H3VR options and the developer statement, and **Knifie_Sp00nie**
whose request made that reasoning public:
<https://steamcommunity.com/app/450540/discussions/0/3183345176717342122/>. **WarpFrog** (Blade &
Sorcery). **Meta** developer blog, *Tracking Technology Explained: LED Matching*. Generalised out of
[`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr) and
[`visceral-re2-vr`](https://github.com/TefMeister/visceral-re2-vr).

## A retail build that shipped its assertions names its own globals

`[inferred-static 2026-09-05]` · found on UE3, which is the only engine it has been checked against.

Finding an engine's global data structures in a stripped retail binary is normally a byte-signature job
— brittle, per-build, and the part every public SDK generator leaves for you to supply. There is a much
cheaper route to try **first**, whenever a build shipped with assertions compiled in.

Most C/C++ engines define their assertion macro with the preprocessor's stringification operator, so
**the text of the asserted expression becomes a string literal in the binary**. UE3's is representative
(`Development/Src/Core/Inc/UnFile.h`) `[reported 2026-09-05, from public source]`:

```
#define check(expr)  { if(!(expr)) appFailAssert( #expr, __FILE__, __LINE__ ); … }
```

Three things follow, and the second is what makes this worth doing:

1. **A symbol you cannot otherwise search for becomes searchable.** A global like UE3's `GObjObjects`
   never appears as a string in ordinary code — but `check( GObjObjects.Num() == 0 )` puts the text
   `GObjObjects.Num() == 0` in the string pool.
2. **The xref lands *inside* the function that touches the global.** The failure call sits in the same
   basic block as the test, so the global appears there as a **direct memory operand**. That is
   qualitatively better than a string that merely mentions a thing: it is a labelled pointer to the
   access site.
3. **`__FILE__` rides along as free confirmation.** The same call passes the source path, so a
   source-file string sits near every hit. It separates a real hit from a coincidence, says which
   assertion you are standing in, and leaks the studio's source-tree layout for every later hunt in the
   same binary.

**The check that tells you whether the route is open at all is the route itself.** Assertions usually
compile out in shipping configurations (`DO_CHECK`, `NDEBUG` and equivalents), so this is not universal
— but finding *any* recognisable assertion text is the test, and a single hit opens the technique for
**every** variable any surviving assertion guards, not only the symbol you searched for. On one 2010 UE3
retail build the string `GObjObjects` appears **seven times** `[measured 2026-09-04, n=1 binary]`, which
is what prompted this: a build that *looks* stripped may still name its internals.

**Three practical rules.** Search for the **symbol name as a substring**, never for a whole expression —
`#expr` preserves the source's own whitespace, so the exact formatting is compiler- and
version-dependent. Inlining duplicates sites, so a hit count is not a count of distinct assertions. And
**⚠️ scan for BOTH encodings**: on an engine whose own string type is wide (UE3's `TCHAR` is
`wchar_t`), engine *names* are UTF-16 while the assertion text the compiler emits is narrow ASCII
`[measured 2026-09-07]` — one project's symbol scored 7 ASCII hits and **0** UTF-16, so a wide-only
scan would have reported the technique unavailable.

### ✅ Executed 2026-09-07 — the claim held, with four independent corroborations

The xref-lands-in-the-accessing-function step was `[inferred-static]` when this section was written,
read off the macro's expansion rather than confirmed in a disassembler. It has now been run
`[inferred-static 2026-09-07]`, and it is worth recording **how** it was confirmed, because that is the
reusable part:

- **Three different assertion expressions, in three different functions, all resolved to the same
  address.** `cmp dword ptr [X+4], 0` is the `Num() == 0` assertion; `mov eax,[X]` followed by
  `cmp dword ptr [eax+edi*4], 0` is the index-is-null assertion; the valid-index assertion uses `[X+4]`
  as its bound. Three assertions agreeing is a far stronger result than one hit.
- **⭐ A fourth corroboration came free from the layout.** The neighbouring global landed exactly 12
  bytes later — `sizeof(TArray)` on 32-bit — as consecutive statics, with both showing `ArrayNum` at
  `Data + 4`. **A structural prediction that the data confirms is worth more than another string hit**,
  because it could have failed.
- **`__FILE__` delivered as advertised**, giving the studio's full build path from its own machine —
  reusable for every later hunt in the same binary.
- One occurrence turned out to be a **decorated C++ symbol name** rather than an assertion string,
  which is a reminder that a raw hit count mixes sources.

**Honest limits that remain.** The macro shape is near-universal in C/C++, but *"most engines
stringify"* is still `[hypothesis]` — one engine's macro was read, not a survey — and the confirmation
above is `n=1` binary.

### ❌ Two corrections, 2026-09-07, both against this section's own framing

`[verified-numerically 2026-09-07]` Filed the same week the section was written, and both are the same
kind of error: a claim about **two tools** stated as a claim about **a family**.

- **"The public SDK generators ship no patterns" is false of the family.** It is true of the two
  generators this section cites. But a third — a fork in the same lineage — ships **filled-in byte
  signatures for six shipped titles**, and nothing in this account's record mentioned it. The
  motivating argument for reaching for assertion strings *"because nobody will give you the address"*
  is therefore weaker than it was written: **check the forks of a tool before concluding the tool's
  ecosystem does not solve your problem.** A generator's upstream shipping `"null"` placeholders says
  what upstream does, not what the community has published.
- **⚠️ A symbol that no surviving assertion mentions is invisible to this technique — and that is not
  evidence of anything.** On the same binary the sibling global returned **0 hits in both encodings**,
  and a session spent effort explaining the anomaly. There was no anomaly: **no public locator for that
  symbol searches for its name at all.** All six working ones scan for a **code pattern** — an absolute
  load of the array's data pointer followed by a scale-4 indexed read — because that is what survives
  when the name never reaches the binary. **Before explaining your own negative, find out what the
  established toolchain actually does**; if nobody searches by name, a name search returning nothing is
  the normal case and needs no theory.

**The transferable shape of both:** this section is a *string-search* technique, and it locates exactly
those symbols that a surviving assertion happens to name. That is a real and cheap win where it lands
— and it is silent, not negative, everywhere else. Pair it with a code-pattern route rather than
treating a miss as a finding. A third route worth knowing when you have one anchor already: published
**adjacency** between engine globals, which turns a located symbol into a short bounded probe for its
neighbours.

Public sources, read online, nothing cloned or copied: **CodeRedModding**'s UE3 source mirror
(<https://github.com/CodeRedModding/UnrealEngine3>) for the macro and the assertion sites — the engine
source is Epic Games'; **ItsBranK**'s `UE3SDKGenerator` (MIT,
<https://github.com/ItsBranK/UE3SDKGenerator>), whose `Configuration.cpp` ships its patterns as the
literal string `"null"` `[verified-live 2026-09-05, n=1 API read]` — evidence about **those two
generators**, not about the family; see the corrections above, where a fork that ships real signatures
for six titles is recorded. Generalised out of
[`enslaved-vr`](https://github.com/TefMeister/enslaved-vr).

### ⭐⭐ Executed again 2026-09-09 — it beat a published byte signature outright, and the byte signature was worse than useless

The section above framed byte signatures as the brittle default this route improves on. That has now
been **measured head to head on one binary, and the byte route did not merely lose — it returned a
confident wrong answer** `[verified-numerically 2026-09-09, n=1 binary]`.

A widely used UE3 SDK skips the unstable `ProcessEvent` vtable index and scans for the function's own
prologue instead. Published shape: `push ebp` / `mov ebp,esp` / `push -1` / `push <scopetable>` /
`push <handler>` / `mov eax,fs:[0]` / `push eax` / `sub esp,0x50` / ... The same function in one 2013
UE3 PC port begins:

```
55 8b ec 6a ff 68 d0 ca 91 01 64 a1 00 00 00 00 50 83 ec 54
```

Two constants differ, and neither is about the function:

| published | actual in this build | why |
| --- | --- | --- |
| **two** `push imm32` | **one** | this build uses the older `_except_handler3` frame, so the handler comes from the scope table rather than a second push |
| `sub esp,0x50` | `sub esp,0x54` | a different local-frame size |

**A scanner built from the published bytes matched exactly ONE function in 23 MB of code, and it was
the wrong one.**

> **A published prologue byte-pattern is not a property of the function. It is a property of one
> compiler's exception scheme and one build's local-frame size** — both of which change per build,
> per compiler version and per optimisation setting, while the function itself does not.

**And its failure mode is the expensive kind: it returns nothing, which reads as *"the function is
absent"* rather than *"this signature is for a different compiler."*** The researcher who trusts the
pattern concludes something wrong about the binary rather than about the pattern.

What survived the test and what did not:

- ✅ The **strategy** transferred completely — the vtable index is not stable across titles, so detour
  the function's own address instead of taking a slot.
- ⚠️ The **invariant** transferred — "an SEH + stack-cookie frame". That is a shape, not bytes.
- ❌ The **bytes** did not transfer at all.

**The assertion route found the same function in ONE pass**: among the functions bearing assertions
from one named source file, the only VIRTUAL one — 1835 `.rdata` vtable slots against **0** for every
other candidate — asserting a named condition at a named source line. That is the three properties
this section already claims, working end to end: compiler-independent (the string literal is data the
compiler cannot rewrite), self-describing (the assertion text names the source file, so you learn
*what* you found), and self-validating (the `__LINE__` immediate is a second independent check that
the match is the intended call site).

**So the ordering is now measured rather than argued:** where a build shipped its assertions, hunt
assertion strings **first** and keep prologue scanning as the fallback for builds with assertions
compiled out — not the other way round.

#### ⛔ A second warning from the same session: the derived number that "corroborates"

The same work briefly derived a `ProcessEvent` vtable index of **64**, which sat neatly between the
two published UE3 values (60 and 67) and therefore *looked like corroboration*. It was an artefact
`[disproved 2026-09-09]`: it came from treating runs of code pointers in `.rdata` as vtables, and
adjacent vtables in that binary abut with no separator, so runs merge and every derived index shifts.

**A derived value that falls plausibly between two published values is the hardest kind of wrong
number to catch, because the plausibility is doing the verification.** Derive it a second way, or do
not quote it.

Generalised from [`enslaved-vr`](https://github.com/TefMeister/enslaved-vr)
(`external-research/topics/2026-09-07b-public-ue3-locators-find-gnames-by-code-pattern-and-one-fork-ships-working-signatures.md`
§5), measured by that project's `/pd` lane 2026-09-09 with no launch. Credit the `unrealsdk` project
and the UE3 SDK-generator community for the published signature and for the skip-the-index framing,
which is the half that held.


## The cheapest control is the case where the correct answer is "change nothing"

`[verified-live 2026-09-05, n=4 flat launches]` Generalised out of
[`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr).

Most corrections a VR mod computes are **zero in some configuration**. A per-eye offset is zero at the
midpoint; a head-relative correction is zero when the head is where the model was baked; a mirror or
scope steering term is zero when the eye is on the axis it was tuned for. That configuration is a free
and enormously specific control: **run it, and any implementation that does something is wrong before
comfort, feel or magnitude is even a question.**

The worked case is a rifle scope. Flat aim-down-sights puts the eye on the bore by construction, so a
correct steering correction must be the identity there. Three candidate rays were tried against that one
control:

| the eye→target ray aimed at | angle from the bore | picture |
| --- | --- | --- |
| the rig's parked placement | **35.3°** | replaced entirely |
| the weapon transform's root (the grip) | **50.1°** | replaced, worse |
| **the scope's own anchor (a joint plus the mount offset)** | **0.7°** | **unchanged — passes** |

**Two wrong rays caught and the third confirmed, in one day, for three flat launches and no headset
time.** Both wrong ones looked entirely plausible in code.

Three things this technique is really made of:

- **⭐ Verify a model's INPUTS before you disprove the model.** Two imaging models had already been tried
  in the headset and written down as disproved. They were not: each derived its direction from the same
  wrong anchor, so the *arc it was fed* was 35° in a configuration where the eye was on-axis — the model
  was never reached. A day of headset conclusions rested on an input nobody had measured. Print the
  intermediate quantity your model consumes, and check it against the case where you know its value.
- **Run the control at the cheapest gate that can express it.** The identity case here exists in flat
  play, so it costs a flat launch and not a headset session. That is the whole economy of the thing: a
  control that lives one gate cheaper than the failure it catches pays for itself immediately. See also
  [make one launch answer many questions](#make-one-launch-answer-many-questions).
- **⚠️ Passing proves the implementation, not the model.** On-axis is precisely where every value of the
  gain constant behaves identically, so the control says nothing about the gain, its sign, or the law
  relating them. It says the code does not corrupt the case it must not touch. Record which of the two
  you have; conflating them is how a "validated" model reaches a headset untested.

The same session supplies the counter-example that makes the rule sharp. A later finding showed the
picture was tied to the **viewing camera** rather than to the steered plane, so the flat control had been
passing for a reason unrelated to the model being right: in flat aim-down-sights the camera *is* on the
bore, which is why every flat steering test agreed with a correct implementation and with a doomed one
alike. **An identity control is a filter, not a proof** — it removes wrong implementations cheaply, which
is worth a great deal, and it removes nothing else.

## When the shipped inventory has nothing big enough, the limit is on borrowing — not on having

`[verified-live 2026-09-06, n=1 launch]` Generalised out of
[`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr).

A recurring move in this library is **borrow an engine-owned resource instead of creating one**: an
allocation the engine already registers with its own pipeline sidesteps the registration problem that
sinks a resource you create yourself. It works, and it has an apparent ceiling — the game ships what it
ships. One project enumerated the shipped render-target inventory, found the largest usable one at
1920×1080, and recorded *"borrowing a bigger target is exhausted"* as a wall.

**It was not a wall. The asset in question was a 64-byte descriptor file.** In that engine an `.rtex` is
a header — magic, version, DXGI format, width, height, a handful of flags and two floats — and the GPU
allocation is made by the engine at load time from those numbers. Writing one from scratch and dropping
it in as a loose file produced `MIRROR SOURCE latched: 2560x1448`, followed by the pipeline's own HDR
upgrade at the new size. The engine honoured a width and height it never shipped, and the framework's
loose-file loader served a path **the game's archives do not contain at all** — it turned out to be a
loader, not merely an override.

**The generalisable shape.** Before recording "the game ships nothing big enough" as a limit, ask what
the shipped thing actually *is*. If the asset is a **descriptor** — a small header the engine reads in
order to size an allocation — then the inventory bounds what you can *borrow* and says nothing about what
can *exist*. The engine-registration advantage survives intact, because the engine still performs the
allocation; you have only chosen the numbers. Assets in this class are commonplace: render-target
descriptors, buffer and pool declarations, some texture headers, streaming and level-of-detail tables.

**⭐ The control that makes an authored asset trustworthy is a round-trip, not a successful run.** The
writer was validated by making it reproduce **shipped files byte for byte** before any novel size was
attempted `[verified-numerically 2026-09-06, n=2 files]`. That is the difference between "my file worked"
and "my writer is correct": a novel asset that loads proves only that the engine tolerated it, while a
byte-identical reproduction proves you understood the format. Do the reproduction first — it is free, and
it fails loudly.

**And the loader's own log is a positive control that your file is on the path the game reads.** A
sibling project deployed a loose texture for a character who was not on screen; the loader **opened it,
and nothing changed** `[measured 2026-09-06]`. That is the ideal shape for a first deployment on any new
asset path — the open is proof of reach, and the absence of a visible change is not a failure. Without
it you cannot tell "the override does nothing" from "the override was never read".

**⚠️ And confirm the identity of the thing you are editing before you measure it.** In that same project
a full morning of texture measurements — slot contents, tiling scales, material assignments — was taken
against the wrong character, because two similar player IDs were assumed to map the obvious way and did
not `[measured 2026-09-06]`. Everything had to be re-measured. An asset ID is a hypothesis until
something in the running game confirms it.

Credit **Ekey** (REE.PAK.Tool, whose published format description made the descriptor readable) and
**praydog** (REFramework, whose loose-file loader is the delivery path).

### ⚠️ When the change's failure mode is invisible, ship a control asset before the real one

`[hypothesis]` on the specific risk (raised 2026-09-07); the reasoning behind it is `[inferred-static]`. Generalised out of
[`visceral-re2-vr`](https://github.com/TefMeister/visceral-re2-vr).

Authoring an asset puts you one step further from the engine than editing a number, and some asset
formats carry a **silent semantic gamble**. The worked case: a single-channel `BC4_UNORM` mask was
authored for a material's detail-map slot. `BC4` samples as `(R, 0, 0, 1)` — so if the shader reads
that mask's `.g` or `.b`, the mask is **zero everywhere** and the effect it gates dies across the
whole material. Which channel the slot reads is not documented publicly, in a corpus that *does*
document channel packing for the neighbouring map types — which is a genuine negative rather than a
gap in the search.

**And that failure looks exactly like success.** The intended visible outcome was "smoother where
the mask says smooth". A mask that reads as zero everywhere delivers smooth everywhere — correct in
the place you are looking at, regressed everywhere you are not.

Two habits, in order of cheapness:

- **⭐ Prefer the lever with no confound, and separate the diagnostic from the fix.** The same effect
  is gated by two plain float properties on the same material. Setting one to zero disables the
  feature with **no texture edit and no channel gamble at all** — and one of the two already ships at
  zero, so it is realistically one number. When you want to know *whether* a thing is the cause, use
  the lever that cannot fail for an unrelated reason; save the authored asset for when you want the
  *fix*.
- **Ship a control asset whose effect is unmistakable, first.** A uniform mid-grey mask discriminates
  the two outcomes in one launch: if the shader reads the populated channel, the effect halves
  everywhere; if it reads an empty one, nothing changes at all. That is the
  [read-back-against-a-known-value habit](#a-read-back-that-returns-the-same-number-under-every-write-is-three-hypotheses-not-one)
  applied to a texture instead of a scalar — and it is the same idea as validating an image tool on
  synthetic offsets before trusting it on real ones.

**The general shape:** before deploying an authored asset whose failure is invisible, ask what the
asset would look like if the engine ignored it, and whether you could tell. If you could not, author
a deliberately extreme version first. An asset that changes the picture obviously is a measurement;
an asset that changes it subtly is a hope.

## A report from the person in the headset is primary evidence

`[verified-live 2026-09-06]` Generalised out of
[`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr).

Two findings from one VR session, worth stating as a rule because the pull in the other direction is
strong when you hold a rich log and the observer has offered a sentence.

**The observer settles questions the instruments could not.** Three flat launches over two days failed to
establish whether one render-target size was sharper than another — the correlator was fighting a noise
floor larger than the effect it was asked to measure. A bigger change plus a human eye settled it in one
look: *"it is way better the quality, if it stayed like this would be great!"*. When an effect is meant
to be **seen**, a person seeing it is a valid measurement and often the cheapest one available. Record it
as `[verified-live … n=1 observer]` and move on.

**⚠️ And telemetry explains a report — it does not overrule one.** In the same session an observer's
verdict was filed as suspect because the log showed the weapon in a pose that would have invalidated it.
The observer corrected it: those samples were the gaps *between* tests, headset resting on the forehead,
and every verdict had been given while looking. The telemetry was accurate and the inference from it was
wrong, because the log recorded what the hardware was doing and not what the person was attending to. **A
report about what the game looked like is primary; the log is context for it.** When the two disagree,
the log has found a gap in its own coverage — start there, not with the report.

The practical fix is a **judging window**: a one-line, log-side gate that says when a verdict is worth
recording, derived from the pose the test actually requires. That project's is *bore within 20° of the
gaze*, and it caught a bad judging window on the night it was written. A gate like that lets telemetry do
the job it is good at — saying *when* a sample counts — without letting it argue with the person about
*what they saw*.

## Proving BOTH eyes render — on a flat monitor, in one launch

`[verified-live 2026-09-07, n=1 launch]` · `[verified-numerically 2026-09-07, R² = 0.99948]`
Generalised out of
[`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr).

You have a stereo shear working in a graphics-API proxy. The picture moves when you change IPD.
**That proves nothing about stereo.** A single mono view shoved sideways does exactly the same thing,
and mistaking a pan control for a per-eye path is the easiest self-deception available in this work.
The question looks like it has to wait for a headset, because a flat monitor shows one image. It does
not.

**The technique: alternate the eye at the frame boundary, then capture a burst.**

- Flip the eye index **once per `Present`** — never mid-frame, so every frame is still entirely one
  eye and still the exact single-eye path everything else already exercises.
- Keep a **flip counter** in the log. This is the whole difference between *"the eye never
  alternated"* and *"the eye alternated and nothing moved"*, and those two need completely different
  fixes.
- Capture ~16 frames as fast as the OS allows and measure each one's horizontal displacement against
  the first.

**If both eyes are real, the frames fall into exactly TWO clusters** — never three, never a
continuum — because every capture lands on either a left-eye or a right-eye frame. On the worked case
that was 12 px of separation at default IPD, and the two-cluster structure held at every setting
tried.

**Measure the displacement by cross-correlating column-mean intensity profiles.** A stereo shear is a
*coherent horizontal translation*, which is exactly what that measures, and it is nearly blind to
everything that is not one. **Mean-absolute-difference is not a substitute:** an earlier session on
the same game scored input routes by mean-luma delta and reported a *working* lever as "no effect",
because animated grass and water put the noise above the signal. Compare
[the masked-correlation trap](#a-hard-edged-mask-makes-phase-correlation-lie-confidently) — same
lesson, opposite tool.

### The two controls that make this evidence rather than a vibe

**1. Run the stereo-OFF control first, in the same live scene.** With stereo off, 16 captures of a
scene containing walking NPCs, drifting fog and idle animation gave **spread 0 px — every frame
`dx = +0`**, correlation 0.984–1.000. Scene animation does not produce a coherent horizontal
translation, so the noise floor here is not "small", it is **exactly zero**, and any non-zero reading
afterwards is signal. **A control that returns exactly zero is worth far more than one that returns
"about 3 px, probably noise"** — and it is the happy opposite of the case where
[the noise floor is the idle animation](#the-noise-floor-is-the-idle-animation-and-it-can-exceed-the-effect).
Which one you get is a property of the measure you chose, so choose the measure that the confound
cannot express.

**2. Validate the tool on synthetic offsets before trusting it on real ones.** Shift one real frame
by known amounts and require recovery:

```
truth  -40  -12   -3    0   +3  +12  +40
meas   -40  -12   -3    0   +3  +12  +40     7/7 exact, corr 1.0000
different-scene control:                     corr 0.4556
```

Now "high correlation" means *rigid translation* with evidence behind it, and the threshold is
defensible rather than guessed.

### Then make it quantitative: sweep IPD and fit

Two clusters prove *two eyes*. Proportionality proves the separation is a **baseline** and not a
coincidence:

| IPD | 6.5 | 12.5 | 18.5 | 24.5 |
| --- | --- | --- | --- | --- |
| cluster separation | 12 px | 22 px | 33 px | 44 px |

`separation = 1.7833 × ipd + 0.108 px`, **R² = 0.99948**, max residual 0.40 px. Proportional, through
the origin, sub-pixel residuals. Four points and a fit are a different class of claim from *"it got
bigger when I pressed the key"*, and they cost about a minute.

### ⭐ The trap that comes with it: an object at the convergence plane looks unsheared

Rendering the eye pair as a **red/cyan anaglyph** makes disparity visible instantly — and on the
worked case it immediately showed the player character with almost **no** fringing while the whole
world doubled around her. That reads exactly like the classic failure in which skinned character
shaders take a different constant register and never receive the shear. It would have been a serious
finding.

**It was wrong, and the discriminating lever is CONVERGENCE, not the eye:**

| convergence | player character | world wall | NPCs |
| --- | --- | --- | --- |
| 98 | **+78 px** | +96 | +109 |
| 300 (default) | **−1 px** | +17 | +17 |
| 915 | **+26 px** | +8 | −5 |

The character's disparity moves a long way, so she **is** sheared — she simply sits near the
convergence distance, because a third-person camera holds the hero at a roughly fixed range.

**Generalisation: zero disparity is ambiguous between "not sheared" and "at the convergence plane".**
Change convergence and re-measure before concluding anything; if the object moves, it is sheared. On
**any** third-person game this will look like a character-shader bug in an anaglyph, and it is not.

### Three smaller traps from the same launch

- **A far field too dark to match is not "zero disparity".** Block-matching a distant street returned
  peaks of 0.19–0.43 — that is *no measurement*. Report it as unmeasurable and pick a better scene;
  do not record it as a depth-invariant result.
- **Check that a counter counts what its name says.** A `draws_fixed` counter turned out to count
  *pixel*-shader fix-texture bindings rather than sheared draws, so it could not corroborate anything
  about the vertex path. The frame and flip counters could, and did.
- **`F12` is Steam's screenshot key.** Harmless to the game, but the Steam toast sits in screen
  captures for about ten seconds and will quietly contaminate image analysis of that corner. Pick
  hotkeys that the platform overlay does not already own.

**Cost: one launch, and no rebuild if the shear already exists.** Control, four IPD settings, a
convergence sweep and an anaglyph took about fifteen minutes of driving, entirely from outside the
process with synthetic input and screen capture. That is a headset question answered at the
[cheapest gate that can express it](#the-cheapest-control-is-the-case-where-the-correct-answer-is-change-nothing).

## A field map that fits every observed byte is not thereby correct

`[measured 2026-09-07]` Generalised out of
[`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr) and
[`visceral-re2-vr`](https://github.com/TefMeister/visceral-re2-vr) — and it is a correction to this
library's own work, filed the day after it was published.

The previous section recommends validating an asset writer by reproducing shipped files **byte for
byte**. That control is real and it is worth taking. Here is precisely what it does *not* buy you.

One project decoded a 64-byte render-target descriptor across six shipped files, reproduced two of
them byte-identically, and wrote down a field map. A day later a **public decode of the same format**
turned up, written against a much wider corpus — and it disagrees:

| field, in order | our decode | the public library `[verified-live 2026-09-07, n=1 source read]` | observed |
| --- | --- | --- | --- |
| 4th dword | DXGI format | `format`, typed as a **`DxgiFormat`** enum — agrees | `29` / `26` |
| 5th, 6th | width, height | width, height — agrees | vary |
| 7th | depth / array size | **`depth`** | `1` |
| 8th | — | **`mipCount`** | `0` |
| 9th | — | **`arraySize`** | `0` |
| 10th | **mip count** | **`ukn1`** — unknown | `1` |
| two `f32` near the end | two unnamed `1.0` floats | ⭐ **`widthRate`, `heightRate`** | `1.0`, `1.0` |

**Both maps fit every byte in the corpus, and they cannot both be right.** Ours implies mip 1 and
array 1; theirs implies mip 0 and array 0. Ours is the more *physically plausible* reading, which is
presumably why it was written that way — and plausibility is not evidence.

⚠️ **Note what this table deliberately does not give you: byte offsets.** The public reader gates the
last three fields on `version >= 5` and skips a further four bytes at `version >= 6`, so a fixed
offset is only correct for one version — and the version is exactly the thing a modder reading one
game's files is least likely to have varied. Quoting the **order** is a claim the evidence supports;
quoting an offset would have been a fourth guess laid on top of three.

**The rule this yields: the fields that are constant across your corpus are exactly the fields your
corpus cannot name.** A byte-for-byte reproduction proves your *layout* — the offsets, the widths,
the total size — and it proves the writer is faithful. It says nothing about the *semantics* of any
field that never varied in the files you looked at. Name those fields only if you can vary them, or
find someone who has.

Three practical consequences:

- **Look for a public decode before naming a field, not after.** Format libraries, 010 Editor
  templates, extractor source and asset-tool plugins routinely carry a field table for exactly the
  file you are staring at. This one cost nothing to find and would have changed what was written.
- **Record disputed fields as disputed, in the tool.** The correction's instruction was to say so in
  the writer's docstring rather than repeat a name that has not been tested — so the next person
  reading the tool inherits the doubt rather than the guess.
- **⭐ An unnamed constant can be a lever you have not noticed.** The two anonymous `1.0` floats both
  decodes recorded turn out to be **resolution scale rates**. That is a second control over the
  allocated size, sitting in plain sight in a file one project had already reproduced byte for byte,
  invisible precisely because it never varied. When a public decode names your anonymous fields,
  read the names for **capabilities**, not just for correctness.

**What survives unchanged**, and it is worth separating explicitly, because a correction that voids
too much is its own failure: every field between `0x18` and `0x30` is a small constant under *both*
candidate maps, and **none of them scales with width or height** — so the practical conclusion that
drove the work ("there is no size-dependent field to break") never depended on which naming was
right. The byte-for-byte reproduction stands, and so does the live demonstration that the engine
allocates an authored, never-shipped size. **Say which half of a finding a correction reaches.**

Credit **kagenocookie** — [RE-Engine-Lib](https://github.com/kagenocookie/RE-Engine-Lib) (**MIT**),
whose published format reader is the wider-corpus decode. Read online via the GitHub API on
2026-09-07; no code taken.

**⚠️ And a postscript that belongs to this section rather than a footnote.** The correction reached
this library secondhand, and re-deriving it from the source found **a third error in the chain**: the
field is named `format` and *typed* `DxgiFormat`, not named `DxgiFormat`, and the offsets that came
with the correction do not survive the version gating described above. So the sequence ran:
a plausible guess, corrected by a better source, relayed with two new small errors, caught by reading
the source. **A correction is a claim like any other and inherits none of its target's scrutiny for
free.**

## 🚨 The framework you inject through applies its transform to EVERY camera — including the one your feature depends on

`[verified-live 2026-09-07, n=1 source read]` on the framework's code · `[inferred-static 2026-09-07]`
on the consequence for the observed symptom. Generalised out of
[`re-village-scope-vr`](https://github.com/TefMeister/re-village-scope-vr).

A VR mod framework's central job is to force the HMD's per-eye view and projection onto the game's
camera. If your feature involves a **second** render — a scope, a mirror, a portal, a security
monitor, a rear-view mirror, a picture-in-picture map — then the question *"does the framework's
override reach that render too?"* is load-bearing, and it is usually not documented anywhere.

**The worked case.** A rifle-scope mod built on a mature RE Engine framework produced a scope picture
that swung with the player's head in VR while behaving perfectly in flat play. Two headset sessions
were spent on the assumption that the *engine's* mirror component followed the viewing camera, and
several imaging models were built and discarded against it.

**Reading the framework's own source settled it in one pass, and the tell was an asymmetry between
two sibling functions** `[verified-live 2026-09-07, n=1 source read]`, in `src/mods/VR.cpp` on master:

- In the **projection**-matrix hook, the guard restricting the override to the primary camera is
  **commented out** — the override therefore applies to every camera the game asks a projection for.
  (The GUI-camera projection hook next to it has the same guard commented out.)
- In the **view**-matrix hook, the equivalent `if (camera != get_primary_camera()) return;` is
  **live**.

So a secondary render receives **the current eye's asymmetric, off-centre HMD projection over its own
non-eye view matrix**. And that combination has a signature worth memorising: **a projection that
changes with head pose sitting on top of a view matrix that does not** is exactly what "the picture
inside moves where I look" looks like. It is not an engine property and no amount of steering the
secondary render's geometry can cancel it.

Corroboration from the same source that the two renders really are distinct: the framework's own
scene-layer helper treats a layer as *not* the main view precisely when it carries a mirror —
`is_fully_rendered() { return is_enabled() && get_mirror() == nullptr && has_main_camera(); }` — and
the framework's author, asked about scopes in that engine, answered *"The way scopes work is they
create a separate scene, yes."* `[reported, issue #698, 2023-03-27]`

### The transferable habits

- **Before blaming the engine for a secondary render's behaviour, read the injector's camera hooks.**
  You are two layers deep — game, framework, your mod — and the middle layer is the one nobody
  instruments. It is public source in most of these projects and costs one read.
- **⭐ Look for asymmetry between sibling functions.** A guard present in one of a matched pair and
  commented out in the other is a far stronger signal than either function read alone, and a
  commented-out guard usually records a bug someone hit from the *other* direction. Grep for the
  guard, not for the feature.
- **⭐⭐ Search the framework's history for the SYMPTOM, not for the API.** The same author hit this
  exact class of problem on a different title and fixed it by **exemption**: a 2023 commit
  (*"VR (RE4): Fix scope not being zoomed in"*) tested the camera's owning GameObject name for a
  `ScopeCamera` prefix and returned early without overriding, commented *"Allows the sniper scope to
  work."* `[verified-live 2026-09-07, n=1 commit + diff read]` A search for "projection override" would
  never have found it; a search for "scope" did.
- **⚠️ And check the fix is still there.** That exemption is **no longer present in current master** —
  a later refactor replaced the per-game preprocessor blocks with runtime game-identity checks, and
  the block did not survive `[verified-live 2026-09-07, n=1 grep of master]`. **A remedy found in a
  framework's history is a design to re-implement, not a feature to enable**, and this is the
  practical form of
  [dating a dependency](#dating-a-dependency-a-fix-newer-than-your-build-is-not-evidence-that-you-are-affected):
  the question is not only "is the fix newer than my build" but "is it in *any* build, still".
- **A branch or a render mode may already exempt your case.** Forks that add multi-pass rendering
  often filter the layer set they duplicate, and a filter that drops mirror-bearing layers restores
  their own projection as a side effect. Which build and which setting therefore change whether the
  bug exists at all — so **record the framework's branch, commit and settings beside every result**,
  or two sessions will compare observations taken under different code.

**⚠️ The one thing this does not do is retire the engine-level question.** Establishing that the
framework overwrites the projection explains the head-tracking swing; it does not establish what the
secondary render would do without it. Keep those two claims separate, and note which of them each
past observation actually tested — several will turn out to have tested neither.

Credit **praydog** — [REFramework](https://github.com/praydog/REFramework), whose public source is
the evidence for every code claim above, and whose 2023 fix is the design worth copying. Read online
via the GitHub API; no code taken.

## Read a public mod for how MANY levers it writes, not just which one

The usual way a public mod is mined is: find the mod that does the thing you want, learn the API
name it uses, stop. **The count is evidence too, and it is usually cheaper to read than the API.**

A mod that writes **two** levers and sets them to **the same value** is telling you, without saying
so, that the engine does not couple them. If one implied the other, the author would not have
written both — they had the game in front of them and you do not.

**The case this came from.** `visceral-re2-vr` planned its movement-speed feature on a recorded
conclusion that RE Engine locomotion is root-motion driven, so clamping the motion layer's playback
rate would scale travel, leg cycle and footstep events **together** — "by construction". Reading the
reference implementation rather than only its API showed the author does not rely on that at all:
the shipping mod pairs the motion-layer speed write with a **return-value hook on the movement
driver's own speed getter**, applying the same factor to both. A second, independent public
implementation of the same feature arrives at the identical pairing, with separate walk and run
factors `[reported 2026-09-09, from source, n=2 independent implementations]`.

**The transferable habits:**

- **Count the write sites before you copy the API name.** One lever means the engine couples the
  rest; two levers at one value means it does not, and your plan needs both.
- **A second independent implementation is worth more than a second source.** Two authors who never
  read each other converging on the same shape is corroboration; two articles describing one mod is
  not. Look for a *trainer*, a *diagnostic script* or a *different game on the same engine* rather
  than another write-up of the mod you already have.
- **"By construction" is a claim, and it is the kind that never gets tested.** It sounds like a
  property of the engine and is usually a property of nobody having checked. When a plan rests on
  one, find the sentence and tag it — `[hypothesis]` until measured is the honest state.
- **⚠️ This is evidence about the authors, not a measurement of the engine.** Two people writing
  belt-and-braces code is strong reason to plan for two levers; it is not proof the single lever
  fails. It downgrades a "by construction" claim to a hypothesis; it does not disprove it.

Generalised from `visceral-re2-vr` (2026-09-09). Credit **Junh2x** and **Namsku**, whose public
repositories are the evidence; both read for structure only, nothing copied.

## A frame-breakdown "graphics study" documents PASSES, not CONVENTIONS

Frame-by-frame "graphics study" articles are among the best public sources this estate has, and
several projects cite them. **They answer a narrower question than they look like they answer**, and
knowing the boundary saves a whole research pass.

**What they reliably give:** the pass inventory and its order, what each pass renders and into which
target, which buffers are read where, and where the UI is composited. That is exactly what
`doom-2016-vr` used one for to establish that DOOM 2016's HUD is drawn to its own target and
composited last — a real answer to a real question.

**What they do not give:** the maths conventions. Depth direction (reversed-Z or not), infinite far
plane, depth format, row- versus column-major, handedness, the layout of the per-view constant
buffer. A capture-based study reads the *API calls and the images*; the projection convention is a
property of code that never appears in either.

**The case this came from.** `doom-2016-vr`'s critical path is a projection convention it has to
guess at, one game launch per guess. A published statement of id Tech 6's depth convention would have
collapsed that guess-space for free. Three named sources were read against exactly that question —
the two best-known DOOM/DOOM Eternal graphics studies and id Software's own SIGGRAPH renderer talk —
and **none of them documents it** `[reported 2026-09-09, n=3 named sources]`. One of the two studies
says outright that it stays high-level by design.

**The transferable habits:**

- **Match the source class to the question class.** Pass inventory, target formats and compositing
  order → a capture-based study. Matrix conventions, near/far handling, precision choices → the
  engine's own field and cvar names, a leaked or open-sourced predecessor, or a live read. Asking a
  study for a convention is not a hard search, it is the wrong shelf.
- **Record the negative with the sources named.** "Not documented" is only useful if the next reader
  can see *which* sources were checked; otherwise it reads as "someone gave up" and gets re-searched.
- **A convention negative argues FOR the read-it-live route** rather than competing with it. When
  the lookup is unavailable, "read what the engine actually holds" stops being the more expensive
  option and becomes the only one.
- **The engine's own vocabulary is the cheap lever nobody checks.** Field names such as a near-plane
  "cram" or a projection "flip" are themselves statements that the engine has explicit opinions about
  those things — worth reading before assuming any convention.

Generalised from `doom-2016-vr` (2026-09-09). Credit **Adrian Courrèges**, **Simon Coenen**, and
**Tiago Sousa & Jean Geffroy**.

## Stability is not identity: a shared constant register can carry two matrices

`[disproved 2026-09-09]` for the assumption; `[verified-numerically 2026-09-09]` for the defence.
Seen on UE3/D3D9, but nothing about it is UE3-specific — it applies wherever a proxy or hook reads
projection parameters out of an intercepted constant-buffer or constant-register write.

A D3D9 proxy intercepted `SetVertexShaderConstantF(StartRegister=0, count>=4)` — documented as the
engine's view-projection matrix — pulled the projection's horizontal scale `p00` out of it, cached
it, and printed it in a periodic log line.

**The cached value was stable to four decimal places across 33,300 frames.** That read as strong
evidence it was the camera's. It was not. **Register 0 was written by more than one matrix**, and
the periodic report sampled whichever wrote it *last* in the frame — consistently a different,
non-camera one. The camera's own matrix, logged once at startup by a separate diagnostic, had a
`p00` **506x larger**.

The cost was a day: a disparity derivation compared measured screen offset against the wrong number,
found it "380x too small", invented a units mismatch to explain the gap, and queued "apply the
505.8x scale factor" as the project's top task. Applying it would have multiplied eye separation by
~506 and then looked like a tuning problem rather than a wrong premise.

> **A value that never changes is evidence it comes from one source. It is not evidence about
> *which* source.**

### The cheap defence: two scale-sensitive shape tests

A world-to-clip matrix `P*V` built from a symmetric projection and a **rigid** view has two
signatures an arbitrary 4x4 does not:

- **`|row3.xyz| == 1`** — row 3 produces `clip.w`, so for a rigid view it is just the view-forward
  direction;
- **`row0.xyz` perpendicular to `row3.xyz`** — row 0 is `p00 * right`, and right is perpendicular to
  forward.

Two square roots and a dot product per write. Two properties make the pair useful rather than
redundant:

1. **A uniformly scaled camera matrix FAILS this, deliberately.** That is precisely the case where a
   scale factor genuinely *is* needed, so it must stay visible instead of being absorbed silently.
2. **`|row0.xyz| / |row3.xyz|` is scale-free** — a uniform `k` multiplies both and cancels — so it
   recovers `p00` *through* an unknown scale. Use the ratio to read the value and the shape tests to
   decide whether the matrix is the camera's.

⚠️ **Known limit, found by a test that first got it wrong:** a matrix with a tiny `p00` and a perfect
shape is a valid **narrow-FOV** camera and passes. The shape test narrows the question; what settles
it is reporting the **range** of values seen. Two camera-shaped matrices three orders of magnitude
apart cannot both be the camera.

### The half that transfers furthest is about instruments, not matrices

- **Never report a cached "the" value for a register several writers share — report the SPREAD.**
  One field labelled `p00=` implied a uniqueness that did not exist. The replacement prints
  `camera=... last=... range=[min .. max]` plus a count of camera-shaped writes, so one launch
  answers *"how many different matrices arrive here?"* instead of quietly answering a different
  question.
- **A diagnostic that prints a number *and* its interpretation is far more useful than one printing
  only the number — and far more dangerous.** The interpretation is what gets read; the number is
  not. This one printed *"the matrix is uniformly scaled by ~1x ... the unit-mismatch hypothesis is
  CONFIRMED"* — self-contradictory on its face, since "scaled by ~1x" **is** "not scaled" — and it
  was believed for a day because the sentence was confident and the number beside it was never
  re-derived. **If a diagnostic states a conclusion, the branch that picks the conclusion deserves
  its own test.**

See also *"The instrument can be the bug"* and *"Prove the test can fail: mutation-check a numerical
verification before trusting it"* — this is the same family, arriving from the reporting side rather
than the measurement side.

Generalised from [`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr)
(`modding-notes/2026-09-09-the-505x-scale-factor-is-a-phantom-two-matrices-share-c0.md`), 2026-09-09,
`/pd`, no launch.

## An identity that the WRONG answer also satisfies: the inverse-pair check finds fields, not major order

`[verified-numerically 2026-09-09]`. A property of perspective projections, not of any one engine.

Proving that some 64 bytes in memory really are an engine's projection matrix — rather than sixteen
plausible floats at a computed address — is much easier when the engine stores `projectionMatrix`
and `inverseProjectionMatrix` **adjacently**, as many do. Multiply them and require the identity.

**It is a genuinely strong test.** Measured against random data, **0 of 200,000 random matrix pairs
pass**, while a real pair passes at every plausible near/far ratio — including reverse-Z with an
infinite far plane, and a 1.3-million-to-one depth range in float32. Multiply in **both** orders and
take the worse error: a coincidence that satisfies one is very unlikely to satisfy the other.

### ⚠️ What it does not tell you, and the algebra that says why

A test written to assert that a **transposed** inverse — the shape a row/column-major mix-up
produces — would be *rejected* instead **passed**. The reason is structural, not a sloppy tolerance.
For the standard form

```
P = [[a,0,0,0],
     [0,b,0,0],
     [0,0,c,-1],
     [0,0,d, 0]]
```

the inverse's lower-right block is `[[0, 1/d], [-1, c/d]]`, and the usual mapping
`c = zf/(zn-zf)`, `d = zn*zf/(zn-zf)` gives **`d = zn*c`**. So when the **near plane is near 1**,
`1/d` is near `-1`, that block is very nearly **symmetric**, and its transpose is almost itself. No
tolerance loose enough to accept a genuine float32 inverse can separate them.

Measured both ways rather than argued:

| near plane | transposed-inverse error | verdict |
| --- | --- | --- |
| `zn = 1`, `zf = 10000` | **1e-4** | indistinguishable — the transpose passes |
| `zn = 0.05`, `zf = 65536` | **19** | clearly rejected |

### What to carry

- **The check identifies WHICH FIELDS you found**, and that is worth a lot: it converts a computed
  address into an identification, and it is the difference between *"these floats look
  projection-ish"* and *"these two adjacent buffers are a matrix and its inverse"*.
- **It says nothing about row- vs column-major, or handedness.** Read those off the numbers — which
  element carries the `-1`, and what the depth row does. **Print the raw matrix and state the limit
  in the output**, not only in a header comment: a tool that prints a bare verdict here quietly
  invites the wrong conclusion.
- **The discriminating power depends on the near plane**, which is not a knob anyone chooses for this
  purpose. If a project ever does need the transpose separated this way, measure in a scene whose
  near plane is far from 1 — but reading the `-1`'s position is simpler and exact.

**The general form, which is why this earns library space rather than a footnote in one project:**
the failure mode is quiet and flattering — the check *passes*, prints something confident, and the
convention is still unknown. Of any consistency check, ask **"which wrong answers also pass?"**
before quoting what it establishes.

Generalised from [`doom-2016-vr`](https://github.com/TefMeister/doom-2016-vr)
(`modding-notes/2026-09-09-read-the-engines-own-projection-instead-of-guessing-it.md`; host tests in
that project's `proxy-vulkan/test/rvtest.c`, 116 checks, 0 failures), 2026-09-09, `/pd`, no launch.

## Enumerate EVERY input config the game ships — and read them, don't write them

`[verified-live 2026-09-09, n=1 launch]` for the finding, and `[verified-live 2026-09-09, n=1 launch]`
for the correction that follows it. Observed on one UE3 title.

⚠️ **The split is a STUDIO habit, not an engine convention — which is exactly why globbing beats
knowing a filename** `[reported 2026-09-10]`. Public UE3 documentation puts key bindings in
`<Game>Input.ini` under `[Engine.Input]`, with `DefaultInput.ini` as the shipped template; the
second file in the case below carries a name and a row syntax that appear nowhere in that
documentation. So **do not go looking for a file with a particular name in another title** — the
transferable part is the glob and the vocabulary grep, and the fact that a studio may put the half
you need somewhere the engine docs never mention.

A project had spent four sessions on camera control and had established — correctly, with five
candidate causes excluded — that its build exposes no developer console. It had read
`<Game>Input.ini` several times. It had **never opened the second file in the same directory**,
which is where that game keeps its *action* bindings; the file it kept reading holds only axes and
aliases.

The second file contained this, in the retail build, shipped, needing no mod and no rebind:

```
KeyBindArray1=(Name="T",  Command="EnterFPSByRS | OnRelease ToggleCloseFollowCamera")
KeyBindArray1=(Name="XboxTypeS_RightThumbstick",  Command="ToggleGhost | OnRelease ToggleCloseFollowCamera |EnterFPS")
```

**A working first-person camera, on the `T` key, the whole time.** Its primary home is a right-stick
click — a controller chord, exactly the case this library already warns keyboard probing will never
discover. What was new is the second half: it was *also* on a plain letter key, and the reason nobody
pressed it is that the file naming it was never opened.

**Three rules already in this library each nearly caught this, and none did:**

1. *"Titles on this engine often reach debug features by a controller chord rather than a key."* True
   here — and it stopped at *"so use a pad"* rather than *"so read where the chords are declared"*.
2. *"A binding surviving in a shipped ini is not evidence the feature is live."* Also true, and it is
   a rule about **not over-trusting** an ini, which quietly discourages reading more of them.
3. *"The console is absent in this build"* was established well — and for about a day it was read as
   *"the game's commands are unreachable"*, which does not follow. **A key binding that names an
   engine command is a command channel with no console in the path.**

### The rule

> **Before concluding a camera or debug feature is absent from a game, enumerate EVERY
> input-related config the game ships and read all of them** — not just the one named
> `<Game>Input.ini`. Glob `*Input*`, `*Control*`, `*Layout*`, `*Bind*`, `*Key*` across **both** the
> live per-user config tree and the game-folder template tree, and grep the union for **feature
> vocabulary** — `FPS`, `FirstPerson`, `Camera`, `Debug`, `Toggle`, `BugIt`, `Stat`, `Ghost`,
> `Physics` — rather than for key names.

The cost is one `grep` over a handful of text files, before any launch. What it found here was the
single most useful capability discovered on that project.

### ⛔ The correction, measured hours later: reading is not writing

The same finding was initially framed as a **channel**, and a sibling note said outright that some
missing-console capability "may be one rebind away". **That part is disproved**
`[disproved 2026-09-09]`.

Three unused commands were bound to three free keys in both copies of the layout file, game closed.
Nothing happened, and the added rows survived in the file afterwards. That is ambiguous — added rows
ignored, or those particular commands absent — so **a command known to work was moved to a new key**:
`G` given the exact command `T` already carried.

```
before   : third-person
after G  : third-person      <- the working command, on a new key
after T  : FIRST-PERSON      <- seconds later, same run
```

**The game reads its shipped layout and ignores rows added to it.** The loading mechanism is not
established `[hypothesis]`.

> **Such a file tells you what the build CAN DO, and sometimes hands you a key that already invokes
> it. Do not assume you can add to it.** Whether a game re-reads that file is a separate question
> with its own answer per title, and *"the rows are still there afterwards"* does not mean they were
> read. **The check costs one relaunch: put a command you have already seen work onto a new key.**
> If it fires, the file is writable; if it does not, you have learned that before building anything
> on it.

**Why the correction is worth as much as the finding.** The failure was not carelessness about the
commands — each individual name was correctly tagged as a lead. It was that the **mechanism** claim
inherited the confidence of the observation sitting next to it. *"This file lists commands beside
keys"* is an observation. *"This file is how commands get bound to keys"* is a claim about who reads
it, and it needs its own evidence. **A verified observation lending unearned confidence to an
adjacent structural claim** is engine-agnostic and worth naming.

⚠️ Everything else in that file — `ChangeCameraMode`, `ToggleCloseFollowCamera`, `TogglePOI`,
`ToggleGhost`, `togglephysicsmode`, `BugItForGameController`, `StatUnitAndStatFPS` — remains a
**lead, not evidence** `[reported 2026-09-09]`. Only one command was actually run.

Generalised from [`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr),
2026-09-09, `/lm`, two launches.


## Sources

- **XIII (2003) VR** (this account) — harness tick sites, the disproved render-path diagnosis, the log-before-the-call habit, and the exclusive-mode DirectInput wall that `SendInput` cannot cross; generalised out of [`XIII2003-vr/engine-research/`](https://github.com/TefMeister/XIII2003-vr/tree/main/engine-research) §9a/§9b; the byte-identity read-only-tree rule from the same dossier (2026-09-02)
- **Psychonauts VR** (this account) — the void-behind-the-player characterisation and measurement method, the camera-matrix identification arithmetic, the double-rotation trap, the unbound-key false negative and the camera-height-is-not-eye-height rule (notes 71–72); generalised out of [`psychonauts-vr/modding-notes/`](https://github.com/TefMeister/psychonauts-vr/tree/main/modding-notes) and [`psychonauts-vr/dev-archive/`](https://github.com/TefMeister/psychonauts-vr/tree/main/dev-archive)
- **Unreal Gold VR** (this account) — the mutation-checked numerical verification, the no-convergence and full-window-2D-layer design notes; generalised out of [`unreal-gold-vr/modding-notes/`](https://github.com/TefMeister/unreal-gold-vr/tree/main/modding-notes)
- **Visceral — RE2 VR** (this account) — the HMD-anchored body float and the pelvis-drop grounding fix; generalised out of [`visceral-re2-vr/modding-notes/`](https://github.com/TefMeister/visceral-re2-vr/tree/main/modding-notes)
- **RE Village sniper scope** (this account) — the argument-encoding silent-zero case, the hook-to-acquire-a-handle pattern, and the posted-window-message input route; and, from the 2026-09-05/06 sessions, the identity control, the authored render-target descriptor and its byte-for-byte round-trip, the observer-is-primary-evidence rule, the three-hypothesis read-back ladder, the masked phase-correlation trap and its idle noise floor, the expiring resource recognizer, the enumerate-don't-guess reflection rule, and the panel-only-affordance trap; generalised out of [`re-village-scope-vr/modding-notes/`](https://github.com/TefMeister/re-village-scope-vr/tree/main/modding-notes) and [`re-village-scope-vr/engine-research/`](https://github.com/TefMeister/re-village-scope-vr/tree/main/engine-research)
- **Visceral — RE2 VR** (this account) — additionally, from 2026-09-07, the never-name-the-string
  fetcher rule, the disputed-field-map correction, the control-asset habit and the hashed-at-one-level
  caution; and the stale-component read-back guard, the
  intra-frame stale joint matrix, the non-refcounted re-entrancy token, the loader-open-as-positive-control
  habit and the wrong-character-identity caution; generalised out of
  [`visceral-re2-vr/engine-research/`](https://github.com/TefMeister/visceral-re2-vr/tree/main/engine-research)
- **Visceral — RE2 VR** (this account) — additionally, from 2026-09-09, the count-the-write-sites
  rule and the "by construction" caution; generalised out of
  [`visceral-re2-vr/external-research/`](https://github.com/TefMeister/visceral-re2-vr/tree/main/external-research)
- **DOOM 2016 VR** (this account) — the source-class boundary of frame-breakdown graphics studies,
  recorded as a searched negative on named sources; generalised out of
  [`doom-2016-vr/external-research/`](https://github.com/TefMeister/doom-2016-vr/tree/main/external-research)
- **Junh2x** — public Requiem movement-speed mod, read for structure only; the evidence that the
  speed feature is a pair of levers rather than one: <https://github.com/Junh2x/RE9-Movement-Speed-Mod>
- **Namsku** — public RE Engine trainer, read for structure only; the second independent
  implementation of the same pairing: <https://github.com/Namsku/re-engine-trainer>
- **Simon Coenen** — *DOOM Eternal — Graphics Study*, read as the closest sibling reference to
  id Tech 6 and cited here for what a study of that kind does and does not document:
  <https://simoncoenen.com/blog/programming/graphics/DoomEternalStudy>
- **MarsyApp** — **Anomaly VR** (STALKER Anomaly), whose own development thread and Boosty posts
  document a per-weapon **secondary-hand IK offset** specifically to stop the two controllers occluding
  each other for the headset cameras. Read online (in Russian), described in our own words; no code or
  files taken: <https://ap-pro.ru/forums/topic/14575-anomaly-vr/> · <https://boosty.to/anomaly_vr>
- **Downpour Interactive** (Onward) and **UploadVR** — the *Virtual Gunstock* mode, the opposite
  solution to the same problem (stop reading the occluded hand rather than move it), and UploadVR's
  report of it: <https://www.uploadvr.com/onward-inside-out-tracking-update/>
- **Meta** — the developer blog post *Tracking Technology Explained: LED Matching*, cited only for its
  own statement that occlusion is among the worst-case controller-tracking scenarios
- **CodeRedModding** (public UE3 source mirror; the engine source is Epic Games') and **ItsBranK**
  (`UE3SDKGenerator`, MIT) — the assertion-macro expansion, and the evidence that the SDK generators ship
  the harness and not the addresses. Read online, nothing cloned or copied:
  <https://github.com/CodeRedModding/UnrealEngine3> · <https://github.com/ItsBranK/UE3SDKGenerator>
- **DOOM (2016) VR** (this account) — the launch-time gate and the date-match-your-evidence point, the line-endings false negative, the `strings` minimum-length trap, the in-process raw-input route, the call-argument-not-a-global switch shape, and the repeated-launch/ASLR sampling trap; generalised out of [`doom-2016-vr/external-research/`](https://github.com/TefMeister/doom-2016-vr/tree/main/external-research) and [`doom-2016-vr/modding-notes/`](https://github.com/TefMeister/doom-2016-vr/tree/main/modding-notes)
- **Alice: Madness Returns VR**, **Alan Wake VR**, **Prince of Persia (2008) VR** and **Burnout
  Paradise VR** (this account) — the third-party-stereo-fix-as-intelligence method, the proxy-export
  completeness rule and its static-vs-dynamic failure modes, and the instrument-can-be-the-bug case;
  and, from 2026-09-07, **Alice**'s frame-alternating both-eyes proof with its zero-spread control,
  synthetic-offset tool validation, IPD fit and convergence-plane trap, and **Prince of Persia**'s
  CRC32 dictionary reaching past the type table into shipped UI name references, and — from the
  2026-09-07 live session — the device-state injector and its four apply rules, the transport
  comparison, the host-testable pure-function shape, the title-screen negative that could not have gone
  positive, and the data-driven camera takeover with its still-frame ambiguity;
  generalised out of each project's `engine-research/` and `modding-notes/` folders:
  [`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr) ·
  [`alan-wake-vr`](https://github.com/TefMeister/alan-wake-vr) ·
  [`prince-of-persia-2008-vr`](https://github.com/TefMeister/prince-of-persia-2008-vr) ·
  [`burnout-paradise-vr`](https://github.com/TefMeister/burnout-paradise-vr)
- **`ai-game-control-profiles`** (this account) — the shared-vtable rule for DirectInput devices of one
  class, and the incident behind it: a hook installed through the mouse firing for the keyboard, mouse
  deltas landing in the key-state array, and three experiments silently invalidated before it was
  noticed. Paired in the text with the opposite failure from `prince-of-persia-2008-vr`:
  [`ai-game-control-profiles`](https://github.com/TefMeister/ai-game-control-profiles)
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
- **praydog** — additionally **REFramework**'s own source, read via the GitHub API on 2026-09-07:
  the asymmetric primary-camera guard in `src/mods/VR.cpp` (commented out for the projection hook,
  live for the view hook), the `is_fully_rendered()` mirror test in `shared/sdk/Renderer.hpp`, the
  2023 `ScopeCamera` exemption commit `20a3ec54` and its removal in the 2026-04-25 refactor, and the
  issue-#698 comment on scopes rendering as separate scenes. No code taken.
- **kagenocookie** — [RE-Engine-Lib](https://github.com/kagenocookie/RE-Engine-Lib) (MIT), whose
  wider-corpus `.rtex` field map corrected this library's own decode and named two fields both of
  this account's decodes had recorded as anonymous constants. Read online; no code taken.
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
