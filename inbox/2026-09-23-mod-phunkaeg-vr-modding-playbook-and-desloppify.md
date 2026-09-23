# Two outside resources Tefa pointed at, 2026-09-23

## 1. phunkaeg / vr-modding-playbook

https://github.com/phunkaeg/vr-modding-playbook — code MIT, prose CC BY 4.0. Last commit 2026-09-20.

A flat-to-VR engineering reference distilled from 10 in-house ports and ~107 external projects:
failure atlas (symptom → cheap test → likely cause), pattern catalog with stable IDs, teardowns
(BioShock VR, IL-2 1946, Virtua Cop 2, FC2VR, FEAR VR), and compiled + tested reference C++ for
rotation/frames, pose pipeline, stereo projection, hook safety and flat-harness noise-floor stats
(`reference/include/vrref/`). `[reported]`

Worth knowing for the library:

- **It already studies our public work.** `sources.yml` lists our psychonauts-vr, manhunt, PoP 2008,
  The Evil Within, XIII 2003, Unreal Gold and visceral-re2-vr repos (under their old six-repo names).
- Its evidence vocabulary (SOURCE / STATIC / LIVE / HEADSET) is close to our claim tags; a harness
  result is explicitly not headset acceptance — same idea as our GATE split.
- It overlaps several of our engines: Dunia/FC2 (two projects, see the far-cry-2-vr inbox drop),
  CryEngine/Prey 2017 (in-house PreyVR — our Prey project is paused), UE2/XIII, id Tech 6/DOOM 2016
  (KHARVOX, Vulkan), RE Engine (REFramework, RE4 hands mod).
- Suggested use: check its `failure-atlas.md` / `symptom-index.md` before diagnosing a new symptom
  on any project, and compare its `reference/` maths against ours when a camera/projection
  question comes up. Credit phunkaeg per our CREDITS rule if a recipe is used.

## 2. peteromallet / desloppify

https://github.com/peteromallet/desloppify — licence "Open Source Native License 0.2" (unusual;
read before bundling anything, running it is fine). Python 3.11+.

A code-quality scanner for AI-written code: finds dead code, duplication, oversized files and
complexity, gives a score, and runs a fix-it loop. Supports C++ (best with `compile_commands.json`)
and Python; Lua only through its generic tier, if at all. `[reported]`

Fits our code-shape rule (800/1,500-line files, named numbers). **Caution for our use:** its default
prompt tells the agent to follow its queue instead of its own judgement and to do large refactors;
that clashes with our move-only, tagged, proven-no-change split rule, and with mod code whose numbers
were tuned in the headset. Use it as a **read-only report** beside `code-shape-scan.py`, not as a
fix loop, and do not install its Claude skill globally. Not yet tried on any of our repos.
