# id Tech 6 — finding a *dormant* native stereo path

**Engine:** id Tech 6 (DOOM, 2016) · **Render API:** OpenGL *or* Vulkan, chosen per-executable ·
**Public VR conversion:** none exists.

This case study is a different shape from the others here. [Creation Engine 2](./creation-engine-2.md)
and [Anvil](./anvil-per-eye-camera.md) are study notes on *someone else's finished public adapter*.
There is no id Tech 6 adapter to study — so this is instead a worked example of the **reconnaissance
step**: what a careful static pass over an unmodified commercial binary can tell you before you write
any hook code, and one specific pattern it turned up that generalizes well beyond this engine.

The findings below are our own first-party static analysis of a legitimately-owned Steam copy,
published in full in our project repos (linked at the bottom). Nothing here is derived from anyone
else's tool or source.

---

## The headline: the engine already contains a stereo-3D renderer, dormant

Plain string inspection of both shipped executables turns up an entire stereo subsystem that no
menu, setting, or public documentation for the game mentions:

- an enum `stereoRenderMode_t` with the values `STEREO_RENDER_OFF`,
  `STEREO_RENDER_LEFT_AND_RIGHT`, `STEREO_RENDER_TOP_AND_BOTTOM`
- a mode value-name list: `topBottomStereo`, `leftRightStereo`, `HDMI3D`, `HDMI3DtwoPlayer`
- and live cvars carrying **the developers' own help text**:

| cvar | the engine's own description |
|---|---|
| `stereoRender_separation` | "world units from center to eyes" |
| `stereoRender_screenSeparation` | "screen units from center to eyes" |
| `stereoRender_guiOffset` | **"shift guis so they don't appear at infinity in HMDs"** |
| `stereoRender_swapEyes` | "swap target buffers for left and right eyes" |
| `multiView_60Hz` | "0 = alternate frame rendering, 1 = render both each frame" |

The engine names **HMDs** in its own cvar help, and `multiView_60Hz` is a plain-language description
of [AFR versus both-eyes-per-frame](../techniques/#stereo-submission-strategies) — the exact
distinction this library documents as a stereo submission strategy, written by the engine's authors.

This is almost certainly **inherited, not written for this game**. id Tech 5 and the Doom 3 BFG
generation had real stereoscopic support, and these names and help strings are of that vintage. It
survived into id Tech 6 as compiled-in but unexposed code.

### The caveat that matters most

The engine's own documentation comment describes its view setup like this: there is normally one
world view, but there will be *"two unique ones in split-screen multiplayer and **two identical ones
in stereo-3D (both centered between the eyes)**."*

Read carefully, that appears to change the plan: the two stereo views are **identical and centered**,
so the eye offset would not be expressed as two different view matrices, and separation would be
applied **downstream** of view setup as a projection or screen-space step — consistent with a
`screenSeparation` cvar existing alongside a world-units one.

**That was this page's published reading for a week, and a later pass over id's own source
contradicts it.** `[corrected 2026-09-01]`

#### The correction, from id's own published source

id Software's GPL release of **Doom 3 BFG Edition** — the previous generation of this same codebase,
first-party primary source — declares its `renderView_t` in `neo/renderer/RenderWorld.h` like this,
verbatim:

```c
idVec3   vieworg;                   // has already been adjusted for stereo world seperation
idMat3   viewaxis;                  // transformation matrix, view looks down the positive X axis
int      viewEyeBuffer;             // -1 = left eye, 1 = right eye, 0 = monoscopic view or GUI
float    stereoScreenSeparation;    // projection matrix horizontal offset, positive or negative based on camera eye
```

Read the `vieworg` and `stereoScreenSeparation` comments together and the id lineage applies **both**
halves of a textbook stereo setup, as two distinct steps — which is exactly why the engine ships
**two** separately-named separation cvars rather than one:

| cvar | the engine's own help text | what BFG's source shows it doing |
|---|---|---|
| `stereoRender_separation` | "world units from center to eyes" | **moves `vieworg`** — a genuine per-eye world-space camera translation |
| `stereoRender_screenSeparation` | "screen units from center to eyes" | **shifts the projection matrix horizontally** — the convergence term |

So the world-space eye offset is **not** downstream and **not** a skew. The view origin is already
per-eye by the time the renderer sees it, and the projection offset is a complementary convergence
step on top.

`viewEyeBuffer` is worth noticing on its own: a single int, `-1` / `+1` / `0`, where **0 means
monoscopic view *or* GUI**. A one-integer eye selector with the GUI as a first-class member of the
same enum is a recognisable shape to go looking for in any engine of this family.

**Independent corroboration on the successor engine.** Helifax's **6DoF** VR mod for DOOM Eternal
(id Tech 7) is publicly described as using *"synced eye, single pass, stereo instancing"* — a
technique that indexes **per-eye view-projection matrices** from the instance ID and cannot be built
on one shared centered view plus a screen-space shift.

#### The tension, kept rather than resolved

Neither source disproves the other, and this page does not pretend otherwise:

- DOOM 2016's own compiled-in comment really does say *"two identical ones in stereo-3D (both
  centered between the eyes)"* — `[inferred-static, 2026-08-26]`, id Tech 6 text about id Tech 6.
- BFG's `renderView_t` comments really do say `vieworg` is already per-eye —
  `[reported 2026-09-01]`, read from id Software's own published GPL release of Doom 3 BFG, so it is
  first-party text rather than a community account, but it is id Tech 4/5 text describing one
  generation earlier.

Plausible reconciliations, none established: the id Tech 6 comment may describe the world-view list
*as constructed*, before per-eye adjustment is applied to each; it may be stale commentary carried
forward with the code; id Tech 6 may genuinely have simplified the path; or "identical" may mean "of
the same scene" rather than "of the same camera".

**What changed in the recommendation.** The earlier consequence — *"an adapter built on it would
likely still need to override at the projection stage"* — is withdrawn as a default. The **view
stage is now the better first bet on this family**, and there is a live result behind that: the
DOOM 2016 project has a working control point writing the `globalViewOrigin` / `Fwd` / `Left` / `Up`
quartet at a single static global, with the world rendering correctly from the displaced position.
Under this reading that is not a workaround for a missing stereo path — it is **the same lever the
engine's own stereo code pulls**, reached from another direction. The projection route has not been
tried at all, and remains the fallback rather than the plan.

**Status: partly answered, and the answer is instructive.** A live console session on the retail
build established that these cvars are **never registered at runtime** — see
[the gating section](#the-gate-a-dormant-path-can-be-real-and-unreachable) below. Strings prove
the code was *compiled in*; they prove nothing about whether it is *reachable*.

---

## What else the static pass established

### The renderer choice is an executable-level fork
The game ships two binaries. `DOOMx64.exe` imports `OPENGL32.dll` and **not** `vulkan-1.dll`;
`DOOMx64vk.exe` imports `vulkan-1.dll` and **not** `OPENGL32.dll`. The API is not switched at
runtime inside one process — each executable links only its own.

This is a genuinely useful shape to recognize. It means the API you target is the one you launch,
and the other API is entirely absent from the address space. It also means a **direct `OPENGL32.dll`
import** exists to proxy, which is about the cleanest injection foothold available on a modern
64-bit game: an `opengl32` proxy sits in the middle of every GL call including buffer swap, with no
inline hooking, no pattern scanning, and no Control Flow Guard concerns.

### The binary names its own camera
id Tech 6 exposes shader constants as named **"renderparms"**, and the whole name table sits in the
binary as plain strings. The camera-relevant portion:

```
viewMatrixX/Y/Z/W            inverseViewMatrixX/Y/Z/W
modelMatrixX/Y/Z/W           inverseModelMatrixX/Y/Z/W
projectionMatrixX/Y/Z/W      inverseProjectionMatrixX/Y/Z/W
mvpMatrixX/Y/Z/W             inverseMVPMatrixX/Y/Z/W
mvpMatrixNoJitterX/Y/Z/W     mvpMatrixLastX/Y/Z/W
viewProjectionMatrixX/Y/Z/W
globalViewOrigin   globalViewFwd   globalViewLeft   globalViewUp
```

Three things to take from this:

1. **Matrices arrive as four separate vec4 renderparms** (`…X/Y/Z/W`), not as one opaque blob — the
   override granularity is per-row/column.
2. `mvpMatrixNoJitter*` alongside `mvpMatrixLast*` is a clear signature of **TAA with a jittered
   projection plus motion-vector reprojection** — the exact hazard documented under
   [temporal effects](../techniques/#temporal-effects-under-afr). Spotting it statically tells you
   which of the two known fixes you'll be choosing between before you ever run the game.
3. The engine's console exposes **`rp <renderParmName> [value]`**, described in-binary as
   *"Displays or modifies a renderparm"*, plus `renameRenderProg` to hot-swap a shader program.
   A built-in read/write window onto the camera constants is an unusually good verification tool —
   you can confirm your understanding of the camera with zero code written.

### There are override-shaped fields already named
The binary's reflection data contains field names including **`explicitProjectionMatrix`**,
**`explicitFov_x` / `explicitFov_y`**, and **`forceIdentityViewMatrix`**. If fields like these are
honored on the main world view, a per-eye projection override may be a *supported engine input*
rather than something to patch in. **Unverified** — but it is the cheapest high-value thing to test.

### The binary ships a reflection database with human comments
Large string regions carry fully-qualified C++ class, enum, and field names *next to the developers'
own written descriptions* — e.g. an explanatory note that for cinematics they often set the near
plane much lower at the expense of distance depth precision. Functionally this is a built-in symbol
source: structurally the same advantage reflection gives UEVR on Unreal or REFramework on RE Engine,
except native to the binary rather than supplied by a tool.

---

## The gate: a dormant path can be real *and* unreachable

The obvious next move after finding those cvars is to set one from the developer console. On this
game that fails, and the reason generalizes.

The retail build boots into a **production mode** (its startup log announces it) and separately
reports **cheat mode off**. Measured on the shipping build:

| probed | result |
|---|---|
| list all cvars | **171** — an engine of this class has thousands |
| list all commands | **40** |
| search cvars for "stereo" | **nothing** |
| the production-mode master switch | **not itself listed** — it cannot be turned off from the console |
| the "enable developer mode" cvar | present, reads 0 … |
| its neighbour, "FatalError rather than enter dev mode" | … reads **1 by default** |

So the stereo cvars are not *hidden*, they are **never registered**. And the switch that would
change that is behind the same gate. The engine even carries a `CVAR_SHIPPINGDISABLED` flag, which
states the mechanism plainly.

**Three transferable lessons:**

1. **"Present in the binary" and "reachable at runtime" are different claims.** A string sweep
   establishes the first and says nothing about the second. Budget a cheap live probe before
   planning around any dormant feature.
2. **Check the neighbours of a switch before flipping it.** The "enable dev mode" cvar looked like
   the way in; the cvar defined immediately next to it turns that same action into a deliberate fatal
   error. One read-only query avoided a crash. When an engine ships a developer gate, assume it also
   ships a tripwire.
3. **A closed door is worth paying for early.** Establishing that the console cannot reach the
   feature took one short session, and converted "maybe we can drive the engine's own stereo path" —
   a plan that would otherwise have shaped weeks of work — into a known constraint before any code
   existed.

What remains open is narrower and better posed: whether the gated cvars are merely hidden or never
*constructed*. If the latter, in-process registration won't resurrect them either, and the dormant
render code would have to be driven directly rather than through its cvars.

### …and then somebody had already published the key

`[reported, 2026-09-01]` Everything above is correct, and one thing was missing from it. A **public
mod for this same game re-adds the hidden console interface on the retail build, without developer
mode**, taking it from **39 commands / 170 cvars to 290 / 6592** — numbers that match the first-party
live measurement above to within one each. It works via a proxy DLL that patches *before the engine
initialises*, and it separately **reimplements** a handful of commands that were stripped rather than
merely hidden.

**That reframes the open question rather than closing it.** Thousands of cvars complete with the
developers' own help text cannot be hand-authored by a modder; they are an enumeration of structures
the binary already contains. The economical reading is therefore **hidden and constructible**, not
absent — which is the good outcome, because it means in-process registration is a live option.
Recorded at `[reported]` confidence: nobody has tested it on the build in question, and "a tool
exists that says it does X" is not "X happened here."

**Two lessons follow, and they generalise past this game.** First: before writing *"the console is
not a route"* into a dossier, spend one search on whether the engine's modding scene has already
built the unlocker — production-gated engines with active scenes usually have exactly one, because
it is the first thing every modder there wants. Second: **such a tool is worth reading even if you
never install it.** This one publishes its command and cvar lists as plain text, which is a free
symbol source complete with help text — and it settled, from a public page, that a renderparm
read/write command and a *set*-view-position command are real named engine commands rather than
strings of uncertain status. Both are directly relevant to a camera hunt on this engine.

The usual caveats apply and are worth stating: tools like this are frequently closed-source and
unlicensed, which makes them **prior art and feasibility proof rather than something to study
line-by-line**; their patches are build-specific, so compatibility is a thing to verify rather than
assume; and if they proxy a DLL you are also proxying, that is a problem you want to find before a
live run rather than during one.

See [Before you build it, check whether the game shipped it](../techniques/#before-you-build-it-check-whether-the-game-shipped-it)
for the generalised form, which also covers the shipped **photo mode** this same game turned out to
have — an ungated, player-facing detached camera that explains why an elevated view on this engine
renders with no culling collapse.

## A method trap worth stealing

The static pass that found all of the above also produced a **confidently-stated wrong conclusion**,
and the cause was a tooling default rather than an analytical error.

Strings were extracted with **`strings -n 4`** — a four-character minimum, which is the default or
near-default in most implementations. That silently discarded every three-character token. The sweep
therefore reported a particular short console command as *absent from the binary*, when it is in fact
registered and working.

**For command- and cvar-name questions, run the sweep at `-n 2` or `-n 3`**, and cross-check against
a live command listing where one is available. Engine console vocabularies are full of short names
(`god`, `rp`, `map`, `fov`, `set`, `bind`), and a default-threshold sweep quietly hides exactly the
ones most worth finding.

## What generalizes

**1. Check for a dormant native stereo path before assuming you must build one.**
This library already noted that a native `openvr`/`openxr` string is a strong signal an engine has a
VR path worth activating — the RE Engine case. id Tech 6 widens that heuristic: an engine can carry a
usable stereo path with **no VR-runtime strings at all**, because it predates the modern runtimes.
Search for `stereo*` cvars and enums, eye/separation/IPD terminology, and split/multi-view render
modes too. Long-lived engine families inherit this kind of code silently across generations.

**2. Read the engine's own doc-comments — they are load-bearing, and they can still be wrong for
you.** The single most consequential claim on this page originally came from a developer comment
compiled into the binary rather than from disassembly, and reading those comments remains one of the
cheapest high-value moves available. Engines with declarative reflection or cvar-help systems often
ship their authors' prose; it is primary-source engine documentation sitting in the file.

**But that same claim is the one this page had to correct.** A compiled-in comment can be stale,
generation-shifted, or describing a narrower scope than it appears to. So sharpen the rule rather
than abandoning it: **where an engine family has a *published* ancestor, check the ancestor's source
before building a plan on the descendant's comment.** id Tech 6 has one — id's own GPL Doom 3 BFG
release — and one pass over it reversed a conclusion that had shaped the recommendation here. That
cross-check is cheap, repeatable, and applies to every long-lived engine family with an open
generation somewhere behind it.

**3. A dormant path is a starting point, not a finished feature.**
Vintage stereo support was built for 3D TVs and shutter glasses, not head-mounted 6DoF. Expect to
inherit the plumbing (two views, two targets, eye swap, GUI depth) and still have to supply correct
per-eye projection — and *all* of the head tracking, which no dormant path of this era provides.

**4. Static recon has a high ceiling.** Renderer API, injection foothold, DRM state, ASLR/CFG
posture, TAA presence, camera-constant names, and an entire dormant subsystem — all established
before launching the game once. On an unprotected binary this is cheap, safe, and repeatable.

---

## Prior art for stereo on this engine (not head tracking)

For context on what already exists publicly: **Vk3DVision** (Helifax) is a Vulkan stereoscopic-3D
driver with a DOOM (2016) fix, which independently demonstrates that per-eye override at the Vulkan
level is achievable on this title. It is closed-source, so it is a **feasibility proof, not something
to study line-by-line**. See [generic drivers](../generic-drivers/#vulkan--vk3dvision) for details
and limits. vorpX's Geometry-3D mode is reported by its own users as no longer working for this game.

**Status update `[checked 2026-09-01]`, and it matters for planning:** the Vk3DVision repository was
**archived by its owner on 2026-03-05** and is now read-only, with **4.25.5** as its final release.
The maintained fix list still shows DOOM (2016) last updated 2025-08-30. The feasibility proof
stands; **no future fixes will come**, so nothing should be planned around the tool keeping pace with
game patches.

**The split between the two id generations is now explicit and worth stating plainly:** on **id Tech
6 the public prior art is stereo-only**, while the **6DoF VR package exists only for DOOM Eternal on
id Tech 7** (a separate *"Virtual Reality"* build, listed at version 0.90). The long-open question of
whether Vk3DVision's VR naming implied real positional head tracking therefore resolves, *for DOOM
2016*, to **no** — that variant was never built for it. Head tracking on id Tech 6 remains entirely
something a conversion has to supply, and that is now an evidenced statement rather than an absence
of evidence.

---

## Update, 2026-09-01: the gate was never the route to the switch

The whole case above is organised around one implicit assumption — that behind the production-mode
gate sits *the thing that turns stereo on*. Reading the published cvar dump in full retired that
assumption `[reported 2026-09-01]`.

**All 6,572 cvars were read.** The four `stereoRender_*` parameters are there, so are
`multiView_60Hz` and `com_production` — and **nothing in the entire list selects
`stereoRenderMode_t`**. That is a second, independent negative to set beside the live
`listCvars stereo` result: the name is not merely unregistered, it does not appear to exist as a
cvar at all.

id's published previous-generation source shows why. The eye is **an argument to the render call**,
not a mode read from a global:

```c
void RB_DrawView( const void *data, const int stereoEye );   // 0 = mono, -1 / +1 = eyes
```

— carried downstream as a first-class `viewEyeBuffer` field on the view object. `[reported
2026-09-01]` for id Tech 4/5; **`[hypothesis]` for id Tech 6**, a generation later. What makes it
more than a guess is that it *predicts the inventory that was measured*: every stereo **parameter**
present as a cvar while no **mode** is, is the signature of a call-site argument.

**So the conclusion this case study reached is unchanged and its lessons all stand — but the prize
behind the door is smaller than it looked.** Winning the gate yields the stereo *parameters*, which
are genuinely useful, and **not** the on-switch, which was never a cvar to begin with. The
`explicit*` override fields are in the same position: the dump read found none of them present as
cvars, so they are renderparms or code-level fields reached by `rp` or a patch, not by typing at a
prompt.

There is a fourth transferable lesson in that, added to the three above:

4. **When a switch cannot be found, question the category before questioning the search.** Weeks can
   go into looking harder for a global that was never a global. The tell is an inventory where every
   *parameter* of a feature is exposed and nothing selects the *mode* — read that as evidence about
   the shape of the code, not about how well hidden the name is. Written up in full on the
   [techniques page](../techniques/#the-switch-you-cannot-find-may-be-an-argument-not-a-global).

---

## Sources

**Our own first-party analysis** (static inspection of an owned copy; full evidence published):
- [`doom-2016-vr/engine-research/`](https://github.com/TefMeister/doom-2016-vr/tree/main/engine-research) — the consolidated engine dossier
- [`doom-2016-vr/dev-archive/`](https://github.com/TefMeister/doom-2016-vr/tree/main/dev-archive) — raw recon evidence (`recon/2026-08-26-phase0-static/`)
- [`doom-2016-vr/modding-notes/`](https://github.com/TefMeister/doom-2016-vr/tree/main/modding-notes) — the session write-up

**Developer-authored material on this renderer:**
- Tiago Sousa & Jean Geffroy, *"The Devil is in the Details: idTech 666"*, SIGGRAPH 2016 Advances in Real-Time Rendering — [slides](https://www.slideshare.net/TiagoAlexSousa/siggraph2016-the-devil-is-in-the-details-idtech-666) · [text summary (80.lv)](https://80.lv/articles/idtech-666-the-secret-of-dooms-render). Describes the renderer as a hybrid clustered-forward + deferred design with roughly 100 unique shaders in total, and notes the id Tech 6 job system's scheduling gaps that id Tech 7 later rewrote.
- Adrian Courrèges, *"DOOM (2016) — Graphics Study"* — [adriancourreges.com](https://www.adriancourreges.com/blog/2016/09/09/doom-2016-graphics-study/). Frame-by-frame breakdown of a real capture; the reference to reach for when doing pass inventory.

**First-party engine source (the correction above):**
- [id-Software/DOOM-3-BFG](https://github.com/id-Software/DOOM-3-BFG) — id Software's own GPL release of the previous generation of this codebase; `neo/renderer/RenderWorld.h` (`renderView_t`) and `neo/renderer/RenderSystem_init.cpp` (the `stereoRender_*` cvars and the `STEREO3D_*` mode enum).

**Third-party tools referenced:**
- [Vk3DVision](https://github.com/helifax/Vk3DVision-Public) (Helifax / Octavian Vasilov) — closed-source, Patreon-funded; releases only. **Archived 2026-03-05**, final release 4.25.5. Per-title fix versions and dates: [3dsurroundgaming.com](https://3dsurroundgaming.com/Vk3DVisionGames.html).
- DOOM Eternal 6DoF VR mod (Helifax) — technique prior art on id Tech 7; reported by [Flat2VR](https://x.com/Flat2VR/status/1704495949978984506), [demo video](https://www.youtube.com/watch?v=6Z-LGvDUlv8).
- [DOOMLegacyMod](https://github.com/brunoanc/DOOMLegacyMod) — **emoose** (original), updated and re-hosted by **brunoanc**. Re-adds DOOM 2016's hidden console commands and cvars on retail; publishes `doom_cmds.txt` and `doom_cvars.txt` as plain-text interface dumps. Closed-source, no licence stated — referenced as prior art and read online only.
- [vorpX](https://www.vorpx.com/) (Ralf Ostertag) — commercial.

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
