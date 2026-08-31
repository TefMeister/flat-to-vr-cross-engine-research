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

### A Lua GUI callback's `false` return was broken for nine days in 2026

`[reported]` `on_pre_gui_draw_element` returning `false` is the documented way to stop an element
drawing, and it is how HUD-hiding scripts across this family work. A refactor merged as
[PR #1503](https://github.com/praydog/REFramework/pull/1503) (2026-08-19) silently broke it; it was
fixed by **ErwinGunsmith** in [PR #1809](https://github.com/praydog/REFramework/pull/1809), merged
2026-08-28. **A REFramework build from that window runs your suppression script without error and
draws the element anyway.** If a HUD-hiding script "does nothing", check the framework revision date
before debugging your own code. The cause generalises — see
[techniques → silent no-ops](../techniques/README.md#silent-no-ops-verification-that-cannot-see-the-failure).

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

## See also

- [engines index](../engines-index.md) — the "Capcom RE Engine" row.
- [techniques](../techniques/README.md) — the engine-agnostic forms of several findings above.
