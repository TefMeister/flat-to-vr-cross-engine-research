# Xbox 360 static recompilation (ReXGlue)

*A route family rather than an engine: console games that never had a PC release, rebuilt as native
Windows programs by static recompilation. This page holds the shared truth; each game's detail lives in
its project's `ENGINE-DOSSIER.md`. Curated by the cross-project research sweep.*

## Identity

- **What it is:** the [ReXGlue SDK](https://github.com/rexglue/rexglue-sdk) translates an Xbox 360
  game's PowerPC executable (`default.xex`) **ahead of time into C++**, which is then compiled as an
  ordinary Windows program. **Not emulation**: there is no interpreter or JIT in the hot path
  `[reported 2026-09-17]`. Public coverage describes Xenia's GPU backend being used as the rendering
  service for now `[reported 2026-09-17]`, which matches the Xenia-shaped class names both our projects
  found (`CommandProcessor`, `PipelineCache`, `GuestOutput`, a GPU module named `xenos`)
  `[inferred-static 2026-09-16]`.
- **Render API:** Direct3D 12 in the builds we run; the SDK also carries a Vulkan backend
  `[verified-live 2026-09-16]`. OpenXR supports both.
- **Known public VR path:** none. The SDK has no OpenXR code, no VR files and no VR issues
  `[verified-live 2026-09-16]`.
- **Legal footing on this account:** the game data comes from the owner's own discs, dumped by the
  owner. Nothing from a disc is redistributed.

## Why this route is different from every other project here

Every other project starts from a sealed retail binary and injects. **Here the whole program is source
code we build**, so stereo can be added in the source instead of hooked in — and **work on the shared
SDK lands on every ReXGlue title at once**.

## Our projects on this route

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| Condemned 2: Bloodshot (Monolith, 2008) — via [Condemned2Recomp](https://github.com/psxrestore/Condemned2Recomp) | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/condemned-2-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`condemned-2-vr`](https://github.com/TefMeister/condemned-2-vr) |
| The Darkness (Starbreeze, 2007) — our own recompilation | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/the-darkness-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`the-darkness-vr`](https://github.com/TefMeister/the-darkness-vr) |

## Shared findings

### Two seams, and only one of them makes depth `[inferred-static 2026-09-16]`

Found by reading the SDK source (v0.10.0) on The Darkness, and shared code, so it applies to Condemned 2:

- **Seam A — output.** `D3D12CommandProcessor::IssueSwap` runs **once per game frame** and holds the
  finished frame as an `ID3D12Resource`; the presenter's `Present()` follows. This is where eye images
  would be copied into an OpenXR swapchain and `xrEndFrame` submitted, with the desktop window kept as a
  mirror. **By then the frame is flat** — sending it to both eyes gives no depth.
- **Seam B — render.** The game uploads its shader constants, **including its view and projection
  matrices**, by writing the emulated GPU's constant registers (indices 0–255 vertex, 256–511 pixel).
  **Every camera matrix therefore passes through one SDK function**, which is where a per-eye offset
  would go.

The `DXGI` swap chain's `Stereo = FALSE` flag is the old quad-buffer 3D-monitor mode and is **not**
relevant to OpenXR — noted so nobody chases it.

### Practical traps met so far

- **The published SDK binaries require AVX2** (Haswell, 2013+); older CPUs die instantly with an
  illegal-instruction fault `[verified-live 2026-09-16]` (Condemned 2). Building from source for the target CPU is
  the way round it.
- **Drive it with a virtual Xbox pad** (ViGEmBus). Title screen to live gameplay was reached that way,
  and look-stick turning was measured by phase correlation on the lower half of the frame, because
  fixed HUD prompts pin a whole-frame match at zero `[verified-live 2026-09-16]` (The Darkness).
- **An idle front end plays an attract trailer** that looks like gameplay. Verbose file logging settles
  it: the trailer opens a video file and no level file `[verified-live 2026-09-16]` (The Darkness).

## See also

- [engines index](../engines-index.md) — the ReXGlue row.
- [Hook to acquire a handle the API will not give you](../techniques/README.md#hook-to-acquire-a-handle-the-api-will-not-give-you) —
  not needed here, which is the point: the handle is in source.
