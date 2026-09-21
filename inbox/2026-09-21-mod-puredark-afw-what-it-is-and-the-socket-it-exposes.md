# PureDark's AFW (Alternate Frame Warping) — what it is, and the socket it exposes

**From:** modding session, home PC (RTX), 2026-09-21. Asked for by Tefa, who pasted the repo link.
**Source read:** `github.com/PureDark/UEVR` — branches `master`, `AFW`, `Joey-Merged`; releases
`UEVR_AFW_v1.0-beta.1` … `beta.6` (2026-07-09 … 2026-08-13); files read in full or in part:
AFW-branch `README.md`, all six release notes, `dependencies/pd-afwmod/include/PDAFWPlugin.h` (349
lines, in full), `src/mods/vr/UpscaleHelper.hpp` (hook names only), and the `master...AFW` compare.
**Engine-agnostic, hence this inbox.** Nothing here has been run by us.

## What it is

A fork of praydog's UEVR (Unreal Engine 4.8–5.4 universal VR injector) that adds a new rendering
mode. PureDark's own words: *"a new rendering method that can probably boost the performance in VR
for about 60-80%"*, *"There could be minor glitches/artifacts"*, *"uses about 500mb extra VRAM"*,
**DX12 only**. `[reported]`

**Wearer report, ours:** Tefa ran it on Silent Hill 2 (remake): *"the performace boost was
incredible with just a little artifacting around the hands and weapons, but a fair tradeff to have
the game running smoothly."* `[reported 2026-09-21, n=1 wearer]` — That artefact is a known,
named one: release note 4 says first-person arms and weapons can show **double vision** and lists
five games fixed by a per-game profile script (A Quiet Place, Atomic Heart, Ghostwire Tokyo, Ready
or Not, Oblivion). SH2 is not on that list.

## How it works, as far as the public files show

- **The game draws less, and the missing pictures are made by warping ones that exist.** The header
  names three modes: `AlternateEyeWarping` (make one eye from the other), `PreviousFrameWarping`
  (make this frame from the last one), `CombinedWarping` (both). `[inferred-static]`
- **A warp needs three things per picture — colour, depth, and motion vectors — plus the camera
  matrices of where the picture was taken and where it should appear to be taken from**
  (`FrameBufferDesc{color, depth, motionVectors}`, `CameraData{src…, dest…}`). `[inferred-static]`
- **It gets depth and motion vectors by listening in on the game's own upscaler call.**
  `UpscaleHelper.hpp` hooks `NVSDK_NGX_D3D12_EvaluateFeature` (DLSS), `ffxDispatch` (FSR 2/3 API)
  and `xessD3D12…` (XeSS), and reads the `Depth` / `MotionVectors` parameters the game hands over.
  That is why the install notes say *"turn on DLSS/DLAA"* first, and why beta 5 added OptiScaler
  support for AMD cards. `[inferred-static]`
  ⇒ **A game with no temporal upscaler gives it nothing to listen to.**
- Extras in the same socket: CAS sharpen, foveated composite, variable-rate-shading image
  generation, UI extraction and re-projection, motion-vector correction (several kinds, including a
  range-limited fix *"for first person view … prefered to be set to 0.5f"*). `[inferred-static]`

## ⭐ The part that matters for us: the warping is a SEALED add-on with a PUBLISHED socket

- The warp itself is **not in the repo.** `dependencies/pd-afwmod/` holds only the header and a
  `dummy/PDAFWPlugin.cpp`; the real thing ships as a compiled DLL in the releases. `[inferred-static]`
- The header's three exported calls are plain D3D12 and **mention Unreal nowhere required**:
  `InitDevice(DeviceParams)`, `InitFrameWarp(FrameWarpInitParams)`,
  `EvaluateFrameWarp(FrameWarpEvaluateParams&)`. The only Unreal-flavoured field,
  `InUEVelocityBuffer`, is optional. `[inferred-static]`
- So in principle **any D3D12 VR mod that can supply colour + depth + motion vectors + camera
  matrices could call it** — it is not welded to UEVR. `[hypothesis]` — nobody here has tried, and
  the DLL may check its host.
- ⚠️ **Licence / terms: NOT ESTABLISHED.** The repo's licence is praydog's; nothing read so far says
  what PureDark allows for the compiled add-on. It is his work: use only with his say-so, credited,
  never copied. Ask before building anything on it.

## Open questions this raises (none investigated — Tefa's call: after the scope is ready)

1. **RE Engine + REFramework:** can the main VR view supply depth and motion vectors? RE Village's
   own upscaler is, as far as I recall, FSR **1** — spatial, no depth or motion vectors — which
   would give the listener nothing. `[hypothesis]` unchecked. PureDark is also remembered as having
   made an upscaler add-on for the RE Engine games under REFramework, which would be the natural
   bridge; **unverified, from memory only.** `[reported]`
2. **A separate picture (the Village scope):** the scope is a single extra view, so "one eye from
   the other" does not apply, but `PreviousFrameWarping` would: draw the scope every other frame,
   warp the frames between. Needs depth + motion vectors **for the mirror pass**, which we have
   never looked for. We already own its camera matrices. `[hypothesis]`
   ⚠️ The scope picture is magnified ~2.4× by cropping, so any warp error arrives magnified too.
3. Whether two independent warp instances (game and scope) can coexist — `InitFrameWarp` takes one
   HMD size and returns one pair of eye buffers, which reads as one instance per process.
   `[inferred-static]`

**Idea this serves:** `mod-ideas` → `games/re-village.md` → Performance → "PureDark's AFW,
switchable separately for the game and for the scope".
