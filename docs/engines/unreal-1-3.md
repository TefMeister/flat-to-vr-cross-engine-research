# Unreal Engine 1–3

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** Epic Games' Unreal Engine, generations 1 through 3 — including licensee layers
  built on top of it (Ninja Theory's NTEngine layer on UE3, for Enslaved).
- **Render API:** UE1 uses pluggable render-device DLLs (OldUnreal's 227k patch adds modern
  renderers and native 64-bit builds); UE2 is Direct3D 8; UE3 is Direct3D 9.
- **Known public VR path:** none turnkey. [UEVR](https://github.com/praydog/UEVR) (praydog)
  attaches only to UE 4.8–5.x and cannot be made to attach below that floor — for these
  generations it is a conceptual reference only (see the canonical playbook's appendix on what in
  UEVR is engine-agnostic and what is UE4/5-locked). Manual build.

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| Unreal Gold (Unreal, 1998, plus its expansion) — UE1 via OldUnreal 227k | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/unreal-gold-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`unreal-gold-vr`](https://github.com/TefMeister/unreal-gold-vr) |
| XIII (2003) — UE2.x | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/XIII2003-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`XIII2003-vr`](https://github.com/TefMeister/XIII2003-vr) |
| Enslaved: Odyssey to the West (Premium Edition) — UE3 + Ninja Theory's NTEngine layer | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/enslaved-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`enslaved-vr`](https://github.com/TefMeister/enslaved-vr) |
| Alice: Madness Returns (2011) — UE3 | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/alice-madness-returns-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr) |

## Shared findings

*Seeded 2026-08-26; grown by the research sweep as cross-project truths emerge. The per-project
dossiers linked above remain the source of truth for game-specific detail.*

### Camera delivery: where the view actually lives, per generation

**UE3 / D3D9 — there IS a shared view-projection, and it is at `c0`.**
`[inferred-static 2026-09-01, n=2 games — and, unusually, from two different kinds of evidence]`

| Register | Contents | Enslaved | Alice: Madness Returns |
|---|---|---|---|
| **`c0`–`c3`** | **`ViewProjectionMatrix`** — world space to projection space | shipped `.usf` **source** | compiled shader **reflection** |
| **`c4`** | **`CameraPosition` / `ViewOrigin`** — the world-space camera position, handed to you directly | shipped `.usf` source | compiled shader reflection |
| **`c5`** | **`PreViewTranslation`** — the far-from-origin precision offset applied to `LocalToWorld` | shipped `.usf` source | compiled shader reflection |

Enslaved ships its UE3 HLSL sources, and `Common.usf` reserves the engine registers explicitly, noting
they must match `EVertexShaderRegister` in `RHI.h`. Alice ships no sources but ships its compiled
shader cache, and reading the reflection tables out of it produces the **identical map** — with no
exceptions across thousands of shaders and hundreds of distinct layouts.

**Why the agreement is worth more than two more source reads would be.** Source is what the engine's
authors wrote; reflection is what the shader compiler actually emitted for shipped content. The two
cannot fail the same way, so their agreement rules out both "the comment is stale" and "this build
diverged". Treat the map as the family default and verify per title rather than deriving it again.

`LocalToWorld` and `PreviousLocalToWorld` are declared in the **vertex factories**, so they are
compiler-allocated and land at whatever higher registers the shader happens to use. That makes
`SetVertexShaderConstantF(StartRegister == 0, Vector4fCount == 4)` a **clean single injection point**
for a per-eye offset, and it means the camera position does not have to be solved out of a matrix.

**⚠️ The trap that comes with it: `PreViewTranslation` means vertices arrive in *translated* world
space.** A per-eye offset that ignores `c5` looks correct near the origin and drifts as you move away
from it — a bug that passes its first test and fails later, far from where it was written.

**🪤 Split reflection tables by shader target before aggregating registers — this nearly produced the
wrong answer.** In Alice the same name `ViewProjectionMatrix` sits at **`c0` in vertex shaders and at
`c4` in pixel shaders**, and the pixel shaders are far more numerous. Aggregate the two together and
`c0` looks like the *minority* case, making the wrong conclusion the obvious one. **Vertex and pixel
constant registers are separate address spaces**; a tool that reports a register without its target is
reporting half a fact. A related display hazard in the same class: tooling that prints *sampler*
registers with a `c` prefix will show `s1` as `c1`, where it collides with a genuine float4 `c1`.

**🪤 Now `n=2` — the sibling project fell into exactly this trap, and the correction matters for its
consequence.** `[inferred-static 2026-09-02, n=34,046 tables; the SM2 cache agrees]` Enslaved's first
reflection read (2026-09-01) reported `ViewProjectionMatrix` at `c0` (3325 tables), `c3` (288) and
`c10` (22), concluded "about 9% of variants read it from `c3`/`c10`", and widened its proxy to accept
those registers. It had not recorded the stage. Re-walked with the version token that sits just before
each `CTAB` block (`0xFFFE` vertex, `0xFFFF` pixel), **every one of the 3325 vertex-shader
view-projections is at `c0`**; the 310 others are pixel shaders. Two things follow:

- **On the vertex side — the only side a `SetVertexShaderConstantF` hook sees — the view-projection is
  at `c0` and nowhere else**, so one per-eye offset at vs `c0` covers every vertex position. The
  widened acceptance was unreachable for the matrix it was written to catch.
- **What that leaves open is the pixel side.** Those 310 pixel shaders project *something* with the
  un-offset matrix — screen-space effects, reflections, decals; the table cannot say which — and a
  vertex-stage hook never touches them. Whether it shows in stereo is a live question; the fix, if
  needed, is the same hook with the same bit-identity guard on `SetPixelShaderConstantF`. `[hypothesis]`

The pixel-side reserved registers seen so far: Enslaved has `ScreenPositionScaleBias` at ps `c1`,
`MinZ_MaxZRatio` at ps `c2` and the view-projection at ps `c3` (a minority at ps `c10`); Alice has its
pixel-side view-projection at ps `c4`. **So unlike the vertex map, the pixel-side slot is not the same
across titles — record it per game, with the stage.** The rule this family now runs on: **the stage
travels with the register in every table and every log line.** "`c3` ×4 equal to `c0`" is one
character from a rule that, on a build carrying NVIDIA's stereo branch, would treat a stereo parameter
as a view-projection. Full correction in
[`enslaved-vr/modding-notes/`](https://github.com/TefMeister/enslaved-vr/tree/main/modding-notes)
(`2026-09-02-viewprojection-c3-c10-are-pixel-shaders-no-nvidia-stereo-branch.md`).

**⚠️ And the reading this corrects is instructive.** A capture had recorded `c0` receiving 47 uploads
per frame and concluded there was no shared view-projection. UE3's D3D9 RHI **re-applies the reserved
view registers around bound-shader-state changes**, so those are 47 writes of the same value. See
[counting events is not measuring content](../techniques/#counting-events-is-not-measuring-content).

**UE2 / D3D8 — everything funnels through one transform setter.**
`[inferred-static 2026-09-01, from XIII]` `FD3DRenderInterface::SetTransform` in the render-device
DLL is where every world, view and projection matrix passes on its way to
`IDirect3DDevice8::SetTransform`; its type argument maps to `D3DTS_WORLD` / `VIEW` / `PROJECTION` and
each is cached on the interface object. Both halves of true stereo in one `__thiscall`, which makes
it the natural per-eye hook.

**⚠️ That setter early-outs on an unchanged matrix**, so a naive per-eye write can leave the second
eye inheriting the first's view and stereo collapses to mono **silently**. Full treatment:
[a setter that early-outs on an unchanged matrix](../techniques/#stereo-hazard-a-setter-that-early-outs-on-an-unchanged-matrix).

**UE1 / 227k — there is no view matrix, so a per-eye camera is a per-eye translation of view space.**
`[verified-numerically 2026-09-02, from Unreal Gold; not yet rendered]` UE1 hands the render device
already-transformed view-space vertices plus a per-frame projection description, never a view matrix.
Unreal Gold's from-scratch D3D11 render device therefore does stereo with **one extra float in its
projection constant buffer** — an eye shift added to view-space X before the unchanged mono projection
— two constant buffers (one per eye), and every world batch issued twice into two non-overlapping
half-width viewports over one shared depth buffer, so the eyes never depth-test against each other.
Mono constants stay bit-identical to the pre-stereo build. Convergence is deliberately *not* modelled:
parallel cameras with identical projections put zero disparity at infinity, which is what a headset
compositor wants (the asymmetric frusta come later from the runtime's eye tangents), even though on a
flat 3D display it puts everything in front of the screen. How this was verified without a launch is
its own technique: [prove the test can fail](../techniques/#prove-the-test-can-fail-mutation-check-a-numerical-verification-before-trusting-it).

**Head-look needs no native code on UE1 or UE2 — the view is produced by a script event.**
`[reported 2026-09-01, UE1, from OldUnreal's public 227 UnrealScript; the UE2 half is XIII's live hook
on the same event, below]` On UE1 the player's view is produced by a script-implemented `event` on the
player pawn that returns the view actor, location and rotation through `out` parameters. The rotation
property it writes is declared `transient`, so driving it every frame at headset rate replicates
nothing and persists nothing; FOV is a plain script float; and the head-bob a VR conversion must remove
is added *inside that event*, in first person only — on this engine killing it is one omitted line. On
UE2, XIII reaches the same event from a native DLL through its exported name. Same seam, two
generations, so the family split is:

| Job | Layer | Native code? |
| --- | --- | --- |
| HMD **orientation** into the view; killing head-bob | script / player layer (`PlayerCalcView`) | no |
| Per-eye **offset** and per-eye **projection** | the render device | it is the renderer you are writing anyway |
| Getting the HMD **pose** *into* script | a native package bound to a script package | yes — the one native piece, and on UE1 a documented community procedure: compile the script package with `ucc make -nobind`, ship a DLL named after the package, and look for `Bound to <Package>.dll` in the log |

Detail and sources in
[`unreal-gold-vr/external-research/topics/`](https://github.com/TefMeister/unreal-gold-vr/tree/main/external-research/topics)
(`2026-09-01-playercalcview-is-a-script-event-so-head-look-needs-no-native-code.md`). Whether that
native-package recipe holds for a 64-bit host is an open assumption there, not a fact.

**Family-wide habit this earns:** UE1–3 titles frequently ship their own `.usf` shader sources or an
unpacked shader cache. **Look for them before planning a capture** — on this family a register map
that a frame capture would answer ambiguously is often written down, by Epic, in the game folder. See
[read the shipped files before you attach anything](../techniques/#read-the-shipped-files-before-you-attach-anything).

### Driving the game from an injected hook (UE2, from XIII)

`[verified-live 2026-08-28, n=2 — two faults, two different call sites]`
**A global `UGameEngine::Exec`-style entry point can be unsafe to call from an injected hook
regardless of which phase you call it from.** In XIII (2003) it faulted from a camera hook inside
`UGameEngine::Draw` *and* — after the render-path explanation was assumed and the tier re-armed —
from `ULevel::Tick`, with no render path in the stack. Prefer **narrowly-scoped dispatch objects**
instead: `UObject::ScriptConsoleExec` on the player controller reaches only that object's own exec
functions, and the cheats live on a separate `UCheatManager` the console reaches by hopping from
the controller. Locate it by **exported-vtable identity** (`??_7UCheatManager@@6B@`) rather than a
hardcoded offset, and re-validate on use — it is destroyed on level change.

Corollary worth checking on any UE1–3 title: **"it is a standard UE command, therefore it is
here" does not hold.** XIII's build has `God`/`Fly`/`Ghost`/`Walk`/`HealMe` but no `Teleport`,
`SetSpeed`, `Slomo`, `Invisible` or `Loaded`. Probe, don't assume.

### UE2's documentation is public; its C++ headers never were — and Epic says VR drivers were built at the render-device layer

`[reported 2026-09-01]` From [`XIII2003-vr`](https://github.com/TefMeister/XIII2003-vr), correcting
that project's own earlier over-broad claim that no public UE2 documentation equivalent exists.

**Split the claim in two, because only one half is true and only the other half blocks anything.**

- **A public, Epic-hosted UE2 documentation set does exist**, still served under
  `docs.unrealengine.com/udk/Two/` — the UnrealScript language reference, a generated index of Runtime
  topics, level-design and **rendering configuration** material, editor documentation, and `UnDox`,
  Epic's own tool for turning UnrealScript into HTML documentation. Worth knowing beside the usual
  community decompilers, and it is the layer at which UnrealScript-level work (decompiling `.u`
  packages to find gameplay functions) actually happens.
- **The C++ native source headers are NDA-gated**, and were at the time. The declarations a native
  camera/projection hook needs — the render-interface class, its transform-type enum and constants —
  have no public reference. So on UE1–2, decoding that interface by hand from the binary is not a
  research failure; **there was never a public document to find.**

**⭐ And the same Epic page supplies unusually good first-party corroboration for the hook layer this
family's projects converge on.** Describing what licensees historically did with those native headers,
Epic lists — alongside motion-capture device interfaces — **"360 degree rendering drivers for VR
systems"**. That is the engine's author describing its own **render-device interface** as the
documented extension point at which VR and multi-view rendering drivers were built, in this engine
generation, by people who had the headers.

Which is exactly where the `SetTransform` hook above sits. A recon result that is `[inferred-static]`
and has never been run is a different proposition when the engine vendor records that this precise
layer is where such drivers were historically built: it does not verify the hook, but it strongly
supports that the layer is sufficient. **The pluggable render-device DLL is not a clever intrusion on
UE1–3 — it is the sanctioned seam.**

**Fetch caveat:** every direct fetch of `docs.unrealengine.com/udk/Two/*` returned **HTTP 403**. Page
titles and the NDA statement are well-attested across search results; treat finer detail as unverified
until someone opens the pages in a browser.

### Input is alias-based, and useful aliases often ship unbound

`[verified-live 2026-08-28]` UE1–3 route input through named aliases (`Axis aBaseX Speed=...`,
`button bUp`) defined in the ini and bound to keys separately. A game can therefore **define an
action it never binds** — XIII ships `TurnLeft`/`TurnRight`/`FastTurnL`/`FastTurnR` with no key on
any of them, which reads as "the engine cannot do this" when in fact only the binding is missing.
Binding a spare key needs **no code change at all**. Check the alias table before building
anything to synthesise an input.

Two traps that cost real time on XIII, both plausibly family-wide:

- **The ini the game writes is not always the ini you should edit.** XIII *deletes* `User.ini` on
  exit and regenerates it at launch from `DefUser.ini`, so edits to the former always vanish and
  the template is the only durable target. Check which file survives a restart before concluding
  a binding "did not take".
- **A rotator read from telemetry may be unwrapped.** XIII's yaw is a raw accumulating integer
  that runs straight through the 65536 boundary. Applying the usual shortest-arc wrap to it turns
  a real −199° turn into an apparent +161°, which looks exactly like an input reversing direction.

### The cheap first milestone on UE2, and exactly what it cannot become

`[verified-live 2026-08-27, Quest 3 via SteamVR]` XIII reached a headset with head-look **without any
camera reverse-engineering at all**, by capturing the finished D3D8 back buffer each frame, presenting
it as a flat SteamVR overlay, and driving the game's own view rotation from the HMD. Two things make
this unusually cheap on UE1–3 specifically:

- **The renderer is a pluggable DLL loaded by name**, so shipping a proxy render device is *idiomatic*
  for the engine rather than an intrusion — no external injector, just a file in `system\`. Generate
  the forwarding exports from the stock DLL's own export table.
- **The view rotation is a single exported function's output parameter.**
  `APlayerController::eventPlayerCalcView` is exported under its decorated name, so the hook needs no
  hardcoded address; call the original, then add the HMD-derived yaw and pitch to the out `FRotator`.
  **Negate the HMD yaw** — OpenVR/OpenXR yaw winds opposite to Unreal's — and remember `FRotator` is
  three `int32` in Pitch/Yaw/Roll order with **65536 units per revolution**, not degrees. Getting that
  conversion wrong is the classic "camera barely moves, or spins wildly" bug.

**Its ceiling is structural:** one image to both eyes means no stereo depth, and because the headset
pose never enters the simulation there is no 6DoF and no motion-controlled aim. On this family the
second milestone is the `SetTransform` hook above. Full treatment of the route, its failure modes and
its ceiling:
[capturing the finished frame](../techniques/#capturing-the-finished-frame-the-whole-frame-route-to-a-headset).

One family-relevant gotcha that comes with it: **the classic UE main loop gates `Engine->Tick` on
`GetForegroundWindow()`**, so an unfocused game stops presenting within about a second — fatal for a
headset. See
[an old main loop may stop rendering the moment it loses focus](../techniques/#an-old-main-loop-may-stop-rendering-the-moment-it-loses-focus).

### ⭐ UE3 shipped an official NVIDIA 3D Vision integration — check for it before building stereo from scratch

`[reported 2026-09-01 — Epic's own UDK documentation, corroborated by the joint Epic/NVIDIA GDC
announcement]` Surfaced by
[`alice-madness-returns-vr`](https://github.com/TefMeister/alice-madness-returns-vr)'s shader-cache
work and confirmed against Epic's published page by the research sweep.

Epic and NVIDIA announced a **stereoscopic-3D integration coded into Unreal Engine 3 itself**, and
Epic documented it under the title *"Unreal Engine 3 and NVIDIA 3D Vision **Direct**"*. Three facts
from that page matter to anyone converting a UE3 title:

- **It is enabled by an ini key — `AllowNvidiaStereo3d=True`.** A config key, not a code change.
- **It works in fullscreen only, and not in the editor.** This is a real constraint on any live test:
  *a windowed test showing nothing is not evidence of absence.* Anyone who has spent an evening
  concluding a stereo path is dead should check they were not windowed the whole time.
- **"Direct" is the mode in which the *application* renders both eyes**, rather than the driver
  duplicating draw calls. If a given build genuinely shipped that path, a UE3 game contains a two-eye
  render path already — the single most valuable thing a conversion can inherit.

**⚠️ But do not assume any specific title has it.** Whether a shipped game carries the integration
depends on the engine version it licensed and on whether the licensee enabled it, and there is a
competing signal pulling the other way: UE3-era "3D Vision Ready" titles also carry the **Automatic**
correction pattern compiled into their shaders — a stereo-enabled branch constant and NVIDIA's
`StereoParmsTexture` companion sampler, which on one title appears in **65% of every pixel shader in
the game**. A texture whose job is to *tell a shader which eye it is drawing* is the signature of
driver-duplicated draws, not of an application that already knows.

**Both can be true of one binary, and the two are separable statically, before anything is launched:**

1. **Read the shipped shader cache.** The stereo branch constant and the stereo-parameters sampler are
   ordinary reflection entries. Their presence establishes 3D-Vision *awareness* and tells you which
   shaders participate — see [read the shipped files before you attach
   anything](../techniques/#read-the-shipped-files-before-you-attach-anything).
2. **Count the callers of the NVAPI mode-setter.** Whether the game ever hands per-eye rendering to
   itself is decided by whether it calls the driver-mode setter before device creation. Zero callers
   where sibling stereo wrappers *do* have callers means Automatic, which means the game's stereo
   symbols are a correction layer and **not** a lever on the camera. Full method, including the trap
   that a bare zero proves nothing: [counting callers separates what a binary links from what it
   uses](../techniques/#counting-callers-separates-what-a-binary-links-from-what-it-uses).

**And `n=2` now says the integration is per licensee build, not per engine generation.**
`[inferred-static 2026-09-02, n=2 titles]` Alice (UE3, 2011) carries the stereo branch in 65% of its
pixel shaders. Enslaved (UE3, same D3D9 generation) has **zero** occurrences of `nvstereo` across all
three `RefShaderCache` packages including SM4, all three `GlobalShaderCache` binaries, the executable
and every shipped `.usf`, and `AllowNvidiaStereo3d` in no file — the only NVIDIA change markers in its
shipped sources are a particle-colour edit. Two consequences for the check: run it as a
**case-insensitive byte search over the caches**, not a name lookup through a reflection parser — it
also covers the SM4 (D3D10 `RDEF`) caches a `CTAB` walker cannot read, and it catches the sampler name
as well as the constant; and treat **an NVIDIA engineer's markers in the source as no evidence of the
stereo branch** — they edited other things too. A clean negative this cheap belongs in the dossier so
nobody re-derives it.

**Either answer is useful, and neither leaves you empty-handed.** Direct means an inheritable two-eye
path behind an ini key. Automatic means the shaders sample a **stereo parameters texture the
application itself creates** — and a proxy that binds its own texture in its place drives separation,
convergence and the per-eye sign across every shader that samples it, with no NVIDIA driver and no
shader patched. That route is written up in full at [taking over the stereo parameters
texture](../techniques/#-taking-over-the-stereo-parameters-texture--the-cost-above-turned-into-a-lever).

**Fetch caveat, stated because it bounds the confidence:** `docs.unrealengine.com/udk/*` returns
**HTTP 403** to automated fetch. The page title, the ini key and the fullscreen-only restriction are
well-attested across search results and the GDC announcement is independently reported, but no session
here has read the page directly in a browser. Treat the three bullets as reliable and anything finer
as unverified.

### ⚠️ UE2 / D3D8: a per-draw stereo route does not automatically cover the whole frame

`[inferred-static 2026-09-02, n=1 binary]` from XIII (2003). The attractive stereo design at the
D3D8 layer is to draw every world batch twice, once per eye, with per-eye transforms and half-width
viewports — attractive partly because it never modifies the render device's cached matrix and so has
no [early-out](../techniques/README.md#stereo-hazard-a-setter-that-early-outs-on-an-unchanged-matrix)
to design around.

It has a gap on this generation, and the gap is measurable rather than theoretical. A whole-`.text`
vtable-call scan of one UE2-era `D3DDrv` found the **programmable vertex-shader path present and
used** — ten shader creations, seventeen binds, four constant uploads, one of which pushes five
consecutive constants from register `c0` — alongside fifteen fixed-function transform calls. Draws
running under a vertex shader take their view and projection from **those constants**, not from the
fixed-function transform state, so a per-draw route that only handles the latter leaves exactly those
draws **mono**.

What static analysis cannot say is whether those draws carry world geometry or only effects, and what
fraction of a frame they are — so the recon has to **count both paths in the same run** and report
the ratio, not just count the path it intends to hook. Method, and the D3D8-specific trap that an FVF
code and a shader handle share one `DWORD`:
[per-draw stereo reaches only the draws that read the transform you hooked](../techniques/README.md#per-draw-stereo-reaches-only-the-draws-that-read-the-transform-you-hooked).

**Worth knowing on the same generation:** NVIDIA's stereo driver rated UT2004 — same UE2 lineage —
"excellent" by intercepting the fixed-function projection per draw, below the engine `[reported]`.
That is encouraging for the route's *feasibility* on titles whose frames really are fixed-function,
and it is not evidence about any particular binary's shader usage.

### UE3 / D3D9: judge stereo with motion blur off, and check the separation units before believing a symptom

`[reported 2026-09-02]` from Enslaved: Odyssey to the West, corroborated by **eqzitara**'s public 3D
Vision fix for the same game (2013), read online — credited in
[`ATTRIBUTION.md`](../../ATTRIBUTION.md), nothing copied.

- **Motion blur must be off** (`MotionBlur=False` in the engine ini, or the in-game option) before any
  stereo judgement on this generation. UE3's motion blur reprojects at the **pixel** stage, using a
  copy of the view-projection that a vertex-constant injection never touches — so a stereo run judged
  with it on is judging an uncorrected pass. Generalised:
  [turn off the post-processes that re-derive the view](../techniques/README.md#turn-off-the-post-processes-that-re-derive-the-view-before-judging-a-stereo-run).
- **Check the unit convention before calling a separation value wrong.** UE3's conventional world unit
  is 1–2 cm, so a separation of 6 units is 6–12 cm — at or above a real IPD. A "subtly wrong depth"
  symptom at a value in that range is more likely an uncorrected pass than a bad number, and in the
  worked case a completely independent fitting method landed on the same magnitude.
- **A public stereo fix's pass list is a ranked watch list.** That fix had to correct shadows,
  crosshairs, effects and menus, and left HUD depth imperfect even so — which matched, thirteen years
  early, this account's own static prediction that a large family of pixel-stage passes would remain
  uncorrected. Shadows are the first thing to check on a UE3 stereo run, not a generic "watch for
  anything odd".
- **`useAutoTiltup` in the chase-camera ini can be turned off** — a shipped comfort switch, in the
  same class as Alan Wake's `-rigidcamera`. Read the ini before writing a hook.

⚠️ **And a correction worth carrying:** this account recorded on 2026-08-24 that no HelixMod /
3DMigoto entry existed for Enslaved. That was **wrong** — the 2013 fix above exists. A negative about
the public record depends on how you searched, not on what exists; re-run the search before letting
one steer a design.

### UE3, 2026-09-03: Alice stores its matrices as COLUMNS, and Enslaved's build stripped exec dispatch

Two same-day findings from the UE3 pair, both with an engine-agnostic form on the techniques page.

- **Alice: Madness Returns — `ViewProjectionMatrix` is `D3DXPC_MATRIX_COLUMNS`** (`mul`/`mad`
  accumulation, no `dp4`), the transpose of Alan Wake's and Prince of Persia's `MATRIX_ROWS`, so the
  identical clip-space formula needs a transposed implementation: `c[i].x += S·c[i].w` for all four
  registers, then `c3.x −= S·C`. Transplanting the row-form code produces a plausible wrong picture.
  `[verified-numerically 2026-09-03, n=54 configurations]` The exe is **SteamStub v3.1**; unpacking
  a copy let the NVAPI caller-count scan run: **Automatic** mode, getters only, no `SetDriverMode`,
  no `SetActiveEye` — the same verdict as Alan Wake, reached independently, and the two pixel-stage
  facts that follow: `ps c4` is wildly overloaded (a blind write would touch ~33,000 shaders), and
  **the shipped pixel shaders already apply NVIDIA's shear** in 4,042 of them, so a second per-eye
  edit at the pixel stage would double-apply it. `[inferred-static 2026-09-03]` The engine gate
  `AllowNvidiaStereo3d` ships `True` inside `NVCHANGE` markers — the stereo support is an
  NVIDIA-supplied patch to UE3, which is why it appears in these licensees and not in stock UE3.
- **Enslaved — a shipped binding is not a live feature.** `[verified-live 2026-09-03, n=6 keys + 1
  chord]` Tilde/Tab/F10 open no console; shipped and added key bindings for `shot`, `FOV`,
  `ToggleDebugCamera` dispatch nowhere (a developer's own `F9 = shot` binding was the control);
  the debug-camera controller chord through a virtual XInput pad also did nothing, while the same pad
  moved the character. The coherent reading is a shipping UE3 build with the console / cheat-manager
  **exec dispatch compiled out** but its input maps left in. Remaining command channels are in-process
  exec from the proxy (static work) or the pad for pad-gated features. Also measured: shadows pass a
  matched-depth stereo test exactly (no swim), and the D3D9Ex `MANAGED` trap is
  [solved prior art](../techniques/README.md#d3d9-to-a-modern-vr-compositor-the-shared-handle-bridge-and-its-two-traps),
  so route (a) D3D9Ex is preferred over `-d3d10`.
- **Unreal Gold (UE1)** — stock gamma is **not a constant**: `Gamma = Brightness × 2`, the default
  `GM_XOpenGL` mode is the identity at the default `Brightness = 0.5`, so stock applies no gamma at
  defaults; the project's hardcoded 2.0 was `[disproved 2026-09-03]` from the SDK's own source. Stock
  DX9 mode swaps the green and blue exponents. Record `GammaMode` and `Brightness` before any A/B.

### UE3 / D3D9, 2026-09-03d: a device `Reset` disarms the constants hook, and the stereo now has a measured surface table

Third autonomous session on Enslaved, first on the home machine; the engine-agnostic form is on the
techniques page as [a D3D9 `Reset` can disarm a device hook](../techniques/README.md#a-d3d9-reset-can-disarm-a-device-hook-silently-and-late).

- **After any `Reset`, `SetVertexShaderConstantF` is never seen again** `[verified-live 2026-09-03,
  n=2 resets]` — one from a resolution change, one from an ordinary checkpoint restart. `Present`,
  the `Reset` hook and the forced-window logic all keep working; the failure lands 120–240 frames
  *after* the reset returns, which points at UE3 re-creating its RHI/device objects post-reset onto a
  path the vtable patch no longer covers `[hypothesis]`, a static check. A relaunch always restores it.
  **Read the `offset` counter in the log before trusting any stereo observation on this title.**
- **The build ignores its own saved resolution at startup** `[verified-live 2026-09-03, n=2
  launches]`: `MonkeyEngine.ini` says 1280×720, `CreateDevice` asks for the desktop size, the options
  menu shows a third value, and only the `Reset` applies the stored one — so until the reset problem is
  solved, a matched-resolution backbuffer and a live stereo are mutually exclusive.
- **Stereo correctness, measured by surface** (phase-correlation on opposite-eye pairs, instrument
  validated on 7/7 synthetic shifts and refusing to report when the stereo was dead): depth-parallax
  monotonic with distance `[measured 2026-09-03, n=2 scenes]`; HUD/ortho exactly 0 by design; shadows
  track the depth gradient; wet floor and caustics clean `[n=4 eye-pairs]`; **reflective water at a
  glancing angle clean — the largest parallax in the frame** `[measured 2026-09-03, n=16 eye-pairs,
  1 scene]`, with the caveat that the magnitude on a strongly periodic texture is soft even where the
  sign and ordering are not. **Decals are untested**, because identifying a true projected decal by
  eye is the hard part. The two tiles that read exactly zero were both HUD — the ortho fix working,
  and the second time on this project that "check what is behind a probe region" has paid.
- **Input on this build:** no keyboard camera binding at all (mouse-look `[disproved 2026-09-03]`;
  `A`/`D` turn the character and the chase-cam trails). A virtual XInput pad drives the camera on both
  machines `[verified-live 2026-09-03]`, and **needs about two seconds after creation before the game
  acts on it** — at one second the scan returned a flat score curve that read as "the camera will not
  turn". An impatient route, not a dead one; the same shape as
  [saturate first, then tune down](../techniques/README.md#saturate-first-then-tune-down--a-too-small-injection-reads-exactly-like-failure).

## See also

- [engines index](../engines-index.md) — the "Unreal Engine 2 / 3" row.
- [OldUnreal](https://github.com/OldUnreal) — community custodians of UE1; their 227k patch is
  the foundation of the Unreal Gold project.
