# Correction: id's own source says stereo separation moves the *view origin*, not only the projection

**Supersedes:** `docs/case-studies/id-tech-6-dormant-stereo.md` § *"The caveat that matters most"* —
the passage reading *"The two stereo views are **identical and centered** — the eye offset is **not**
expressed as two different view matrices. Separation is applied **downstream** of view setup, as a
projection/screen-space step"*, and the consequence drawn from it (*"an adapter built on it would
likely still need to override at the projection stage rather than assume the view stage will carry
an eye offset"*). Also touches the same page's "What generalizes" item 3.

**From:** `/gr doom-2016-vr`, 2026-09-01 (dev PC)
**Also filed to:** `doom-2016-vr/engine-research/inbox/` — the identical superseded claim lives in
that project's `ENGINE-DOSSIER.md` §6a. Two documents, two owners, not a duplicate.

## The evidence

`neo/renderer/RenderWorld.h` in **id Software's own GPL release of Doom 3 BFG Edition** — first-party
primary source — declares `renderView_t` with these comments, verbatim:

```c
idVec3   vieworg;                   // has already been adjusted for stereo world seperation
idMat3   viewaxis;                  // transformation matrix, view looks down the positive X axis
int      viewEyeBuffer;             // -1 = left eye, 1 = right eye, 0 = monoscopic view or GUI
float    stereoScreenSeparation;    // projection matrix horizontal offset, positive or negative based on camera eye
```

Read the `vieworg` and `stereoScreenSeparation` comments together and the id lineage applies **both**
halves of a textbook stereo setup, as two distinct steps — which is precisely why the engine ships
**two** separately-named separation cvars rather than one:

| cvar (as named in DOOM 2016's binary) | engine's own help text | what BFG's source shows it doing |
|---|---|---|
| `stereoRender_separation` | "world units from center to eyes" | **moves `vieworg`** — a genuine per-eye world-space camera translation |
| `stereoRender_screenSeparation` | "screen units from center to eyes" | **shifts the projection matrix horizontally** — the convergence / screen-plane term |

`viewEyeBuffer` is worth the library's attention on its own: a single int, `-1` / `+1` / `0`, where
**0 means monoscopic view *or* GUI**. A one-integer eye selector with the GUI as a first-class member
of the same enum is a recognisable shape, and worth adding to the "what to look for" heuristics.

## Independent corroboration on the successor engine

**Helifax** (author of Vk3DVision) built a **full 6DOF VR mod for DOOM Eternal** — id Tech 7 — and
the technique is publicly described by Flat2VR as *"synced eye, single pass, stereo instancing."*
Single-pass stereo instancing indexes **per-eye view-projection matrices** from the instance ID. It
cannot be built on one shared centered view plus a screen-space shift. Somebody achieved per-eye view
transforms on the next generation of this engine family.

## Why the original caveat was reasonable, and what should replace it

The case study's caveat came from a doc-comment compiled into DOOM 2016's own binary — *"two
identical ones in stereo-3D (both centered between the eyes)"* — and reading engine doc-comments as
load-bearing is exactly the method the page (rightly) recommends. The method was sound; this is a
second primary source pointing the other way.

**Suggested replacement framing, keeping both readings and tagging each:**

- DOOM 2016's own binary comment — `[inferred-static, 2026-08-26]`, id Tech 6 text about id Tech 6.
- BFG's `renderView_t` comments — `[verified from published first-party source, 2026-09-01]`, id
  Tech 4/5 text, one generation earlier.

Plausible reconciliations, none established: the id Tech 6 comment may describe the world-view list
*as constructed*, before per-eye adjustment is applied; it may be stale commentary carried forward
with the code; id Tech 6 may genuinely have simplified the path; or "identical" may mean "of the
same scene" rather than "of the same camera".

The generalisable lesson survives, but it should be **sharpened rather than deleted**: a
compiled-in doc-comment is primary-source engine documentation and is cheap to read — *and it can
also be stale, generation-shifted, or describing a narrower scope than it appears to.* Where an
engine family has a **published** ancestor, check the ancestor's source before building a plan on the
descendant's comment. That is a cheap, repeatable cross-check the library can recommend generally,
and it is what turned this one up.

## What it changed downstream, for context

On the DOOM 2016 project this reframes a working result. That project has a live control point
writing the `globalViewOrigin` / `Fwd` / `Left` / `Up` quartet at a single static global, with the
world rendering correctly from the displaced position. Under this reading that is not a workaround
for a missing stereo path — it is **the same lever the engine's own stereo code pulls**, reached from
another direction, and a per-eye IPD offset along the basis's `left` vector is the operation the
engine performs on itself.

## Prior-art status change for the same page's "Prior art for stereo on this engine" section

`helifax/Vk3DVision-Public` was **archived by its owner on 2026-03-05** and is now read-only, final
release **4.25.5**. The maintained fix list still shows DOOM (2016) last updated **2025-08-30** and a
separate DOOM Eternal *"Virtual Reality ver. 0.90"* last updated **2024-08-30**. So: the feasibility
proof stands, but no future fixes will come. And the split is now explicit and worth stating on the
page — **on id Tech 6 the public prior art is stereo-only; the 6DOF VR package exists only for id
Tech 7 / DOOM Eternal.** The page's "neither is confirmed to provide true positional head tracking"
line can become a firmer statement for id Tech 6 specifically.

## Sources

- [id-Software/DOOM-3-BFG — official GPL source release](https://github.com/id-Software/DOOM-3-BFG)
  — `neo/renderer/RenderWorld.h` (`renderView_t`), `neo/renderer/RenderSystem_init.cpp`
- [Flat2VR — Helifax's DOOM Eternal 6DOF VR mod](https://x.com/Flat2VR/status/1704495949978984506) ·
  [demo video](https://www.youtube.com/watch?v=6Z-LGvDUlv8)
- [Vk3DVision-Public releases (archived 2026-03-05, final 4.25.5)](https://github.com/helifax/Vk3DVision-Public/releases)
- [VK3DVision game-fix list](https://3dsurroundgaming.com/Vk3DVisionGames.html)
- Full topic write-up:
  [`doom-2016-vr/external-research/topics/2026-09-01-id-own-source-says-the-view-origin-is-moved-per-eye.md`](https://github.com/TefMeister/doom-2016-vr/blob/main/external-research/topics/2026-09-01-id-own-source-says-the-view-origin-is-moved-per-eye.md)
