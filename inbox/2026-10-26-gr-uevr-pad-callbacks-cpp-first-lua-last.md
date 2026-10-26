# /gr: in UEVR, C++ plugins edit the pad before Lua scripts, and Lua always edits last

From: `/gr`, 2026-10-26 (estate sweep), generalised out of `silent-hill-2-remake-vr`.

**Finding** `[inferred-static 2026-10-26]`, read in UEVR's public source (praydog's master and PureDark's AFW fork
carry the same lines): every `on_xinput_get_state` callback, C++ or Lua, sits in one list that each pad poll walks
front to back, each callback editing the same `XINPUT_STATE`. C++ plugins register during `uevr_plugin_initialize`
at start-up, in file-name order (the loader keeps plugins in a sorted map). The Lua side registers on UEVR's first
frame and is removed and re-appended at the end on every "Reset scripts". So on any UEVR game: **C++ plugins see the
real controller first, Lua has the last word**, and a C++ plugin cannot stay after Lua by load order alone.

**Why it transfers:** any project that mixes a native UEVR plugin with a community Lua profile (the common shape:
port a Lua profile to C++ piece by piece) will hit "one press does two things" while both still touch the pad. The
rule that falls out: when C++ takes over input, switch the Lua profile's pad writes off; do not try to undo them.

**Corroboration from a live run:** in the Silent Hill 2 remake, the UEVR MCP plugin's simulated pad was applied after
our plugin's callback (`uevr_mcp` sorts after `sh2vr`) `[verified-live 2026-10-24]`, as this ordering predicts.

**Where it is written up:**
<https://github.com/TefMeister/silent-hill-2-remake-vr/blob/main/external-research/topics/2026-10-26-uevr-runs-cpp-plugins-before-lua-on-the-pad.md>.
Credit praydog (UEVR) and PureDark (AFW fork). Suggested home: `docs/runtime-layers/` (UEVR) or `docs/techniques/`.
