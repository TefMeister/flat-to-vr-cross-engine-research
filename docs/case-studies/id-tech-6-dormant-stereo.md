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

Read that carefully, because it changes the plan. The two stereo views are **identical and centered**
— the eye offset is **not** expressed as two different view matrices. Separation is applied
**downstream** of view setup, as a projection/screen-space step (consistent with a
`screenSeparation` cvar existing alongside a world-units one).

**Consequence:** a dormant path of this vintage may give you correct *stereo* without correct
*per-eye positional geometry*. That is genuinely valuable — real binocular depth, for free, from
code the engine's own authors wrote and shipped — but it is not automatically 6DoF, and an adapter
built on it would likely still need to override at the **projection** stage rather than assume the
view stage will carry an eye offset.

**Status: partly answered, and the answer is instructive.** A live console session on the retail
build established that these cvars are **never registered at runtime** — see
[the gating section](#the-gate-a-dormant-path-can-be-real-and-still-unreachable) below. Strings prove
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

**2. Read the engine's own doc-comments — they are load-bearing.**
The single most consequential fact here (two *centered* views, separation applied downstream) came
from a developer comment compiled into the binary, not from disassembly. Engines with declarative
reflection or cvar-help systems often ship their authors' prose. It is primary-source engine
documentation sitting in the file, and it is cheap to read.

**3. A dormant path is a starting point, not a finished feature.**
Vintage stereo support was built for 3D TVs and shutter glasses, not head-mounted 6DoF. Expect to
inherit the plumbing (two views, two targets, eye swap, GUI depth) and still have to supply correct
per-eye projection — and *all* of the head tracking, which no dormant path of this era provides.

**4. Static recon has a high ceiling.** Renderer API, injection foothold, DRM state, ASLR/CFG
posture, TAA presence, camera-constant names, and an entire dormant subsystem — all established
before launching the game once. On an unprotected binary this is cheap, safe, and repeatable.

---

## Prior art for stereo on this engine (not head tracking)

For context on what already exists publicly: **Vk3DVision** (Helifax) is a maintained Vulkan
stereoscopic-3D driver with an actively-updated DOOM (2016) fix, which independently demonstrates
that per-eye override at the Vulkan level is achievable on this title. It is closed-source, so it is
a **feasibility proof, not something to study line-by-line**. See
[generic drivers](../generic-drivers/#vulkan--vk3dvision) for details and limits. vorpX's Geometry-3D
mode is reported by its own users as no longer working for this game.

Neither is confirmed to provide true positional head tracking; both are best understood as stereo
output rather than a 6DoF conversion.

---

## Sources

**Our own first-party analysis** (static inspection of an owned copy; full evidence published):
- [`doom-2016-vr/engine-research/`](https://github.com/TefMeister/doom-2016-vr/tree/main/engine-research) — the consolidated engine dossier
- [`doom-2016-vr/dev-archive/`](https://github.com/TefMeister/doom-2016-vr/tree/main/dev-archive) — raw recon evidence (`recon/2026-08-26-phase0-static/`)
- [`doom-2016-vr/modding-notes/`](https://github.com/TefMeister/doom-2016-vr/tree/main/modding-notes) — the session write-up

**Developer-authored material on this renderer:**
- Tiago Sousa & Jean Geffroy, *"The Devil is in the Details: idTech 666"*, SIGGRAPH 2016 Advances in Real-Time Rendering — [slides](https://www.slideshare.net/TiagoAlexSousa/siggraph2016-the-devil-is-in-the-details-idtech-666) · [text summary (80.lv)](https://80.lv/articles/idtech-666-the-secret-of-dooms-render). Describes the renderer as a hybrid clustered-forward + deferred design with roughly 100 unique shaders in total, and notes the id Tech 6 job system's scheduling gaps that id Tech 7 later rewrote.
- Adrian Courrèges, *"DOOM (2016) — Graphics Study"* — [adriancourreges.com](https://www.adriancourreges.com/blog/2016/09/09/doom-2016-graphics-study/). Frame-by-frame breakdown of a real capture; the reference to reach for when doing pass inventory.

**Third-party tools referenced:**
- [Vk3DVision](https://github.com/helifax/Vk3DVision-Public) (Helifax / Octavian Vasilov) — closed-source, Patreon-funded; releases only.
- [vorpX](https://www.vorpx.com/) (Ralf Ostertag) — commercial.

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
