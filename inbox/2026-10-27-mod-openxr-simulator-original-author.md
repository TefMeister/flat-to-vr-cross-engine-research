# OpenXR-Simulator: elliotttate wrote the original; fholger's is a fork

Supersedes: docs/runtime-layers/README.md lines 97-101 (fholger named as the original author, fork chain fholger → elliotttate → webhead2oo9)
Supersedes: docs/watch-list.md line 22 (same fork chain)
Supersedes: ATTRIBUTION.md line 276 (links fholger's repo first, elliotttate's not at all)

**What was found** `[measured 2026-10-27]`, from the GitHub API (`gh api repos/<owner>/OpenXR-Simulator`):

- `elliotttate/OpenXR-Simulator`: created 2025-08-09, `fork: false`. This is the original.
- `fholger/OpenXR-Simulator`: created 2026-03-31, `fork: true`, parent `elliotttate/OpenXR-Simulator`.
- `webhead2oo9/OpenXR-Simulator`: a fork of elliotttate's, the one that adds the Vulkan backend, the measured
  headset profiles, real `xrEndFrame` timing, the 32-bit runtime and the MCP server.

So the chain is elliotttate (original) → webhead2oo9 and fholger (both forks). The library had it the other way
round, with fholger first.

**What to change (for /sr):** in the three places above, name elliotttate as the original with a link to
<https://github.com/elliotttate/OpenXR-Simulator>, and webhead2oo9 and fholger as forks. fholger's other credits
(vrperfkit, the Far Cry and Crysis VR mods, Alan Wake VR) are unaffected.

**Already fixed elsewhere:** the toolkit's TOOLKIT.md, CREDITS.md and tools/openxr-probe/README.md (commit 9da0597
in flat-to-vr-RE-toolkit, 2026-10-27).
