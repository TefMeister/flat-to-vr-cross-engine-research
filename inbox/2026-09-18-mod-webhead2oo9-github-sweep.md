# webhead2oo9's GitHub: headset-free OpenXR testing with an MCP server, and four transferable patterns

**From:** modding session, dev PC, 2026-09-18. User-requested sweep of <https://github.com/webhead2oo9/>
(46 repos; the account behind the Virtual Desktop community's Discord tooling).
**Supersedes:** nothing. **Extends:** `inbox/2026-09-17-gr-fholger-openxr-simulator-mit.md` — same tool,
two forks further downstream, with capabilities that note does not mention.

All claims below are `[reported 2026-09-18, from repo READMEs]` unless tagged otherwise. **Nothing here
has been built, run or measured by us.**

---

## 1. OpenXR-Simulator, downstream fork — the headset-free runtime, but further along

Chain: **fholger** (the one already in our inbox) → **elliotttate/OpenXR-Simulator** → **webhead2oo9's fork**
(<https://github.com/webhead2oo9/OpenXR-Simulator>, MIT, last pushed 2026-08-25).

What this fork has that the fholger note does not record:

- **Vulkan backend** alongside D3D11 / D3D12 / OpenGL.
- **Headset emulation profiles** — "measured per-eye FOV, panel resolution and IPD" of ten headsets
  (Quest 2 / 3 / Pro, Index, Vive Pro 2, Reverb G2, PS VR2, PICO 4, Bigscreen Beyond). This is the part
  that matters for stereo maths: a projection bug that only shows at a particular FOV/IPD becomes
  reproducible at a desk, and reproducible *per target headset*.
- **Real XR timing, not window repaints** — title-bar FPS/frametime measured from stereo `xrEndFrame`
  submissions; `F3` gives rolling p50/p95. A frame-pacing number that is not contaminated by the
  desktop present path.
- **32-bit DLL + manifest**, and per-process activation via `XR_RUNTIME_JSON` so the machine-wide
  runtime (Virtual Desktop / SteamVR / Quest Link) is never disturbed. Relevant: several of our targets
  are 32-bit.
- **D3D12 path uses GDI presentation deliberately**, to avoid hook conflicts with the Steam overlay.
  Worth remembering as a symptom signature if our own overlay/hook stack ever fights a runtime.

**The limit, stated plainly:** this is an *OpenXR runtime*. It only helps a mod whose VR output goes
through OpenXR. It does nothing for a mod that renders stereo itself into the game's own swapchain,
which is what most of our per-game work currently does. Its value to us is therefore conditional on
choosing OpenXR as the output path for a given game `[hypothesis]`.

## 2. An MCP server for that runtime — the agent-facing half

`mcp-server/` in the same fork (Python, `openxr_simulator_mcp.py`, with a ready
`claude_mcp_config.json`). Tools it advertises:

- `capture_screenshot` — the current XR frame, `both` / `left` / `right` eye.
- frame diagnostics (timing, resolution, formats), session/head-tracking state, log reading,
  automated diagnosis of common OpenXR faults.
- **quad-layer continuity inspection, separate from world/projection motion** — i.e. "is the menu
  flickering?" answered independently of "is the world moving?".

Why this is the single most interesting thing on the account for us: it is a worked example of
**giving the agent its own eyes on a VR frame**. Today our `/lm` and `/ms` lanes spend the user's
human senses on exactly these questions — is the right eye black, are the two eyes converged, is the
HUD flickering. A per-eye screenshot the session can take itself, at a known emulated IPD, is the
missing sense. The pattern generalises beyond OpenXR: the same shape (MCP over our own injected code,
returning per-eye captures + frame stats) would apply to a custom-stereo mod too.

## 3. The `probe` pattern — a pass/fail test for "will the VR plumbing hold?", with no game

`probe/xr_probe.cpp` + `run_xr_probe.ps1`, written for **BetterVR** (a Cemu/BotW VR layer):
a standalone program that **replays that application's exact OpenXR sequence** — the three extensions
it demands unconditionally, its graphics binding, both swapchain formats, its action shapes, and a
30-frame loop running both of its real frame shapes (quad-only boot/title, then projection+chained
depth+quad in game). **Exit code 0 means the application should run.**

The transferable idea, engine-agnostic: *extract the contract a mod needs from the runtime, and make it
an exit code.* We repeatedly discover a broken assumption only after a headset is on. A probe that
replays our own mod's required call sequence converts a class of `GATE: VR` failures into a `GATE: PD`
check. Directly aligned with our existing rule that a negative result is only evidence if the test
could have produced a positive one.

Also in `BETTERVR.md`: the install **symlinks** the built DLL into the consuming project so a rebuild is
live with no reinstall — with the honest trade-off written down (if the game or probe still holds the
DLL, the *build* now fails at link time instead of the install failing later). That is the standard of
note-writing we want in our own dossiers.

## 4. VirtualDesktopSwitcher — ship the boring fallback route on a branch

<https://github.com/webhead2oo9/VirtualDesktopSwitcher> (C#, 20★, the account's most-starred repo).

Technique: injects a payload DLL into `VirtualDesktop.Streamer.exe` via **EasyHook**, loads a worker into
the process's **default managed AppDomain**, and writes the live .NET setting
(`StreamerSettings.Default.PreferredCodec`) the UI itself uses — so the change applies immediately and
the target window never takes focus. Tested against Streamer 1.34.18 / EasyHook 2.7.7097.

Two things worth taking:

- **A managed-process recipe.** Our injection work is almost entirely native. "Reach the app's own live
  managed setting rather than driving its UI" is the right first move for any .NET target, and
  EasyHook + default-AppDomain is a concrete route.
- **The fallback is a first-class deliverable, on a named branch** (`uia-powershell`), driving the same
  UI through Windows UI Automation for users whose antivirus blocks injection. This is our own
  "build several input routes and measure which one the game obeys" rule, applied to *deployment*
  rather than input: when the clever route is fragile for reasons outside the code, the dull route
  ships beside it, documented, not as a fallback buried in a comment.

## 5. Agent-workflow patterns from the non-VR repos

His own (non-fork) projects carry `CLAUDE.md` / `AGENTS.md` files that are unusually good, and the
style is copyable into our dossiers:

- **Gotchas are stated as consequences, not rules.** "If you add an editor button, add its ID to those
  arrays or the router won't dispatch it." "These normalization quirks are deliberate and locked by
  tests — don't 'fix' them."
- **Characterization baselines + cross-language golden vectors.** Kryten's similarity engine is
  bit-compatible with Python `difflib` and a checked-in vector set enforces it; a directory digest
  "must stay bit-identical everywhere it's computed" and has a Python cross-check. Same idea as our
  claim tags: pin the behaviour so a refactor cannot silently change a result.
- **`dry_run` defaults to `true`** for anything that enforces — detection runs and metrics increment,
  nothing acts, until it is explicitly turned off.
- **One doc for humans, one for agents, with an explicit rule to update both** (UltiMedia:
  `CONTRIBUTING.md` ↔ `CLAUDE.md`).
- **One responsibility per file**, with a table in the agent doc mapping file → responsibility. That is
  our 2026-09-17 code-shape rule, independently arrived at.
- `pi-side-thread` — in-process child agents forked from a live session's last completed reply, so a
  question can be asked *during* a long parent task without polluting its context. Conceptually near
  our one-helper `/lm` shape.

## 6. Leads worth a later look, not investigated here

- **`xrizer`** (fork of `Supreeeme/xrizer`, Rust) — a from-scratch OpenVR implementation on top of
  OpenXR. A reference for how OpenVR calls map to OpenXR when retrofitting VR onto an older game.
- **`ml-sharp`** (fork of Apple's) — "sharp monocular view synthesis in less than a second". A possible
  route to a second eye for content we can never get real depth from: prerendered cutscenes, 2D layers,
  video. Speculative and unmeasured `[hypothesis]`.
- **`ps2_emuvr` / `emu-pages` / `UltiMedia`** — libretro cores and EmuVR work; in-VR emulation rather
  than flat→VR, but the EmuVR override/config plumbing is a documented example of per-folder frontend
  overrides.

---

**For the curator:** §1–§3 belong with the existing fholger note (same tool, better downstream fork) —
suggest they fold into one page under `docs/runtime-layers/`. §4 and §5 are engine-agnostic technique,
not runtime knowledge. Nothing here has been verified against a real build by us; every item is a lead.
