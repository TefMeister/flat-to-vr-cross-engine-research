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
  the way round it. **Update 2026-10-26:** upstream Condemned2Recomp's 0.3.1 release notes say it "no longer requires
  CPUs with AVX2" `[reported 2026-10-26]`, so the trap now belongs to the older releases; check the notes of the one
  you take. (Estacado, a ReXGlue port of The Darkness, still lists AVX2 as required.)
- **Drive it with a virtual Xbox pad** (ViGEmBus). Title screen to live gameplay was reached that way,
  and look-stick turning was measured by phase correlation on the lower half of the frame, because
  fixed HUD prompts pin a whole-frame match at zero `[verified-live 2026-09-16]` (The Darkness).
- **An idle front end plays an attract trailer** that looks like gameplay. Verbose file logging settles
  it: the trailer opens a video file and no level file `[verified-live 2026-09-16]` (The Darkness).

### The Darkness, 2026-09-18: two findings that transfer

From `the-darkness-vr` (dossier, 2026-09-18) `[verified-live 2026-09-18]`:

- **Put the eye offset in the camera, not the projection.** Per-light scissor rectangles, culling and
  the lighting eye position are computed on the host from the unshifted camera, so a projection-only
  offset lit two heads differently per eye; moving it into the camera removed the defect over 21 frames
  (different scene moment, so suggestive). General form in `techniques/` → "An eye offset inside the
  projection matrix is invisible to every decision the CPU makes".
- **Alternate-eye through a recompile needs the world held still for the second eye.** With the
  simulation running, a pair's two eyes were ~130 ms apart in a moving car. A clock pin that holds the
  guest's time on the second frame is built and engaging; the pair rate is then half the guest frame
  rate. See `techniques/` → "Anything that must match between the eyes must advance once per FRAME".

**Condemned 2's engine family has public VR prior art on PC:** three LithTech Jupiter EX mods (DR-89's
fear-vr, and condemned-vr and FEAR2VR as recorded in phunkaeg's playbook). The recompiled binary shares
no addresses with them, but fear-vr's rule — the smallest safe second-eye hook sits below all client
updates and above the pure world render — is a question to ask of the recompiled frame
`[hypothesis]`. Detail: `condemned-2-vr/external-research/topics/2026-09-23-three-vr-mods-on-the-same-lithtech-jupiter-ex-engine.md`.

### Condemned 2, 2026-10-15: the world unit from the game database, and an off-centre view for free

From [`condemned-2-vr`](https://github.com/TefMeister/condemned-2-vr) (dossier §8a):

- **World unit = 1 cm** `[inferred-static 2026-10-15]`, read from the game database (`database\Yeti.gamedb` inside
  `Layer01.Arch02`): player box half-sizes (40, 95, 40), walk 235/s, gravity -2000/s², jump 800/s; only
  centimetres fit every value. The method should carry to the other Xbox 360 Jupiter EX titles (F.E.A.R. 2,
  F.E.A.R. Files) `[hypothesis]`: an `.Arch02` reader, an XMem LZX decompressor (libmspack), and the database's
  name hash `h = h*919 + code(c)` (a-z/A-Z 1..26, 0-9 27..36, `_` 38) `[verified-numerically 2026-10-15, n=9]`.
- **The engine's set-view call already takes an off-centre picture:** after the two FOV tangents comes a second
  float pair, the picture centre in tangent units (`ndc = (tan - centre) / half-width`). A headset's uneven per-eye
  lens fits exactly with half-widths (L+R)/2, (U+D)/2 and centre (R-L)/2, (U-D)/2, with no matrix patching
  `[verified-live 2026-10-15, n=1]` (OpenXR simulator). The general lesson is in `techniques/` → "The engine may have
  no projection matrix to patch".

### 2026-10-26: the SDK's own seams for seeing and driving a game, the first real headset run, and the route moving

**SDK seams, read in the ReXGlue SDK source** by [`the-darkness-vr`](https://github.com/TefMeister/the-darkness-vr)
(dossier, 2026-10-23) `[inferred-static 2026-10-23]`; they hold for every game built on the SDK:

- **Frame pictures without the window:** the presenter can read back the guest's latest finished frame through its
  own command list and fence, from any thread (the xenia screenshot path). It never touches the desktop window, so it
  avoids stale screen copies. Used live by our session tooling on the home PC (title → menu → level, pictures from
  the engine's own frames) `[verified-live 2026-10-24, n=6 runs]`.
- **An extra pad, from the port itself:** the input factory can be replaced before set-up, so a port can add one more
  input driver. A device marked synthetic feeds player one without taking a pad slot, so a real pad keeps working;
  states are merged. Menus that read keystrokes need the driver to queue key events too.
- **Focus:** nothing in the SDK pauses on focus loss; only the real-pad drivers care. A driver living inside the port
  is read whatever window is in front.
- **Building one game rebuilds a shared SDK checkout** when its tree changed, carrying every patch applied there into
  the next game's build. Keep one SDK tree per game, or list the SDK patches in each build's notes.

**The first real headset run** (The Darkness, home PC, 2026-10-25): the per-eye OpenXR patch from Condemned 2 applied
cleanly and ran 42,300 headset frames with no draw errors `[verified-live 2026-10-25, n=1]`, but the head turn curved,
the eyes were off sideways and up-down, and the controls lagged. On the side-by-side build the two halves had looked
in clearly different directions while the bookkeeping counters were perfect `[verified-live 2026-10-24, n=1]`. Cause
open; the checklist in `techniques/` → "What each eye must own when the scene is drawn twice" lists the usual
suspects.

**The route is moving under us** `[reported 2026-10-26, from the repos' own pages, via our /gr lane]`:

- **portingpete's *The Darkness Recomp*** (GPL-3.0, v0.1.7, 2026-10-14) renders **through the engine's own D3D11
  backend** from original geometry and shader sources and "does not interpret console GPU packets"; the engine then
  uses one display size for both its perspective projection and its CPU visibility checks. A port like this exposes
  the camera and projection as ordinary D3D11 state, which may be a cheaper place for stereo than the emulated GPU
  path `[hypothesis]`.
- **invinceble55-wq's *Estacado*** (MIT, ReXGlue, v0.9.8 pre-release) added temporal upscaling with the scene drawn
  jittered at the final composite and the HUD kept separate, so the place where the scene projection can be offset
  per frame has been found in a ReXGlue build by someone else.
- **psxrestore's Condemned2Recomp** reached 0.4.0 (2026-10-24) with a cheat manager and weapon spawner, and lists a
  **native renderer** as in the works. Our headset output and per-eye view are SDK patches in the emulated GPU path,
  so a native renderer would strand them; do not take an upstream update blind.

The lesson for the family: a recompiled game's renderer is not fixed. Check the upstream port's plans before
investing in a seam that the port may replace.

## See also

- [engines index](../engines-index.md) — the ReXGlue row.
- [Hook to acquire a handle the API will not give you](../techniques/README.md#hook-to-acquire-a-handle-the-api-will-not-give-you) —
  not needed here, which is the point: the handle is in source.
