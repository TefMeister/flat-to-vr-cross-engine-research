# phunkaeg's VR Modding Playbook — what it is, and how to use it beside this library

**Source:** phunkaeg, *VR Modding Playbook* — <https://github.com/phunkaeg/vr-modding-playbook>
(code MIT, prose CC BY 4.0; last commit 2026-09-20; ledger dated 2026-09-14). Read online, 2026-09-23.
Everything below is our summary in our own words; nothing is copied.

## What it is `[reported]`

An engineering reference for turning flat games into VR, distilled from about ten in-house ports and
over a hundred external projects. At the time of reading it held 169 named patterns, 397 failure rows
and around a thousand anchors, with its own consistency checks. The parts:

| Part | Use it when |
| --- | --- |
| `docs/symptom-index.md` and `docs/failure-atlas.md` | Something looks or behaves wrong. Symptom → a cheap test that tells causes apart → likely cause → recipe. **The best first stop.** |
| `docs/pattern-catalog.md` | You need a recipe; patterns have stable IDs (e.g. STR-011, HAND-007, CAM-014). |
| `docs/cross-engine-map.md` | Starting a new engine: the problems every engine forced, and the decisions that are engine-specific. |
| Teardown chapters 13, 15–17 | BioShock VR, IL-2 1946 VR, Virtua Cop 2 VR, Far Cry 2 native stereo (plus F.E.A.R. and CryEngine). |
| `reference/` and appendices A1–A5 | Compiled and tested maths: rotation frames, pose pipeline, stereo projection, hook safety, flat-harness statistics. |
| `sources.yml` | Its ledger of every project studied, with engine, API, stereo rung and review depth per area. |

## How it lines up with our own conventions

- **Evidence vocabulary.** It grades claims as `SPEC`, `SOURCE`, `STATIC`, `LIVE`, `HEADSET`, `AUTHOR`
  and `INFERENCE`. Rough map to our tags: `STATIC` ≈ `[inferred-static]`, `LIVE` ≈ `[verified-live]`,
  `AUTHOR` ≈ `[reported]`, `INFERENCE` ≈ `[hypothesis]`. It has no separate grade for "measured
  numerically"; we have no separate grade for `HEADSET`. Its rule that **a harness or simulator result
  is not headset acceptance** is the same split as our `GATE: FLAT` versus `GATE: VR`.
- **It studies our public work.** Its ledger lists our psychonauts, manhunt, Prince of Persia 2008,
  The Evil Within, XIII, Unreal Gold and visceral-re2-vr repositories, **under their old pre-2026-08-30
  six-repo names**, so those links may point at repos that are now frozen duplicates.

## The three ideas most worth carrying into every project

1. **The stereo ladder** (cross-engine map, part 2): R1 the engine renders twice itself (needs a
   re-entrant world-execute step and a camera rebuild you can call — Far Cry 2 reached it); R2 per-draw
   replay (needs a hookable draw stream and reachable per-draw matrices); R3 alternate-eye (needs only a
   frame boundary); R4 reconstruction from the 2D draw stream (Virtua Cop 2). **Age does not predict the
   rung**; how the renderer is reached does.
2. **How the camera reaches the renderer decides a whole bug family** (chapter 17). A camera held as a
   global you overwrite (F.E.A.R., Far Cry 2) needs borrow-and-restore and brings every restore bug; a
   camera passed as a parameter (CryEngine, Quake 2) lets you build a second one and pass it, and the
   family disappears. Check the function that consumes the camera before designing anything.
3. **Walk the shared-problems list as a checklist on day one** (cross-engine map, part 1): which camera
   each consumer uses; projection is a contract of several coupled constants, not one matrix; the engine
   owns culling; mono screen-space buffers break per eye; secondary views; drive the engine's own
   systems; one input path per control; yield to scripted cameras; build and config must identify
   themselves; measure, do not theorise; tonemapping at the eye copy; full-eye FOV; three tracked points
   do not define a torso.

## Where it touches our projects (from the 2026-09-23 `/gr` pass)

| Our project | What the playbook pointed at |
| --- | --- |
| `borderlands-goty-vr` | **Mastersellz's BL1GOTYVR — a working, headset-tested VR mod for the same build.** |
| `bulletstorm-vr`, `enslaved-vr`, `alice-madness-returns-vr` | UE3 seams from BL1GOTYVR (see `engines/unreal-1-3.md`). |
| `condemned-2-vr` | Three LithTech Jupiter EX VR mods (fear-vr, condemned-vr, FEAR2VR). |
| `mad-max-vr` | vaas993's theHunter: Call of the Wild VR, native per-eye on the Apex engine. |
| `re-village-scope-vr` | theHunter VR's measured scope design and flicker post-mortem. |
| `tomb-raider-2013-vr` | farmerarmor's DeusExHRVR, which drives a Crystal-engine game's own HD3D stereo. |
| `far-cry-2-vr`, `far-cry-3-blood-dragon-vr` | Chapter 17 and two Far Cry 2 VR projects. |
| `prey-2017-vr` | The author's own PreyVR, a code read of jordicalsinabaldoma's prey-vr, and a worked CryEngine second pass. |
| `doom-2016-vr` | A code review of KHARVOX. |
| `portal-vr`, `visceral-re2-vr`, `witcher-2-vr`, `death-stranding-vr` | Prior art we already track. |

The rest of the estate had no matching entry. Per-project detail lives in each repo's
`external-research/topics/2026-09-23-*`.

## Cautions

- Its in-house ports are **unreleased**, so their claims can be read but not checked against a build.
- Many external entries are `AUTHOR` grade: the playbook author read the project's README, not its code.
- It is a snapshot. Re-check anything load-bearing at its source.

## Adjacent, not VR: desloppify

Found the same morning: **peteromallet's desloppify** (<https://github.com/peteromallet/desloppify>,
"Open Source Native License 0.2", Python 3.11+), a code-quality scanner for AI-written code. Our estate
uses it **read-only**, as a report beside our own code-shape scan, never through its fix loop — its
default instructions push large refactors, which clash with our move-only, proven-no-change split rule
and with numbers tuned in the headset.
