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

## See also

- [engines index](../engines-index.md) — the "Unreal Engine 2 / 3" row.
- [OldUnreal](https://github.com/OldUnreal) — community custodians of UE1; their 227k patch is
  the foundation of the Unreal Gold project.
