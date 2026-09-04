# Avalanche Engine

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** Avalanche Studios' in-house Avalanche Engine (the Just Cause / Mad Max lineage).
- **Render API:** see the dossier for the measured specifics of the Mad Max build.
- **Known public VR path:** none turnkey. Manual build.

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| Mad Max (2015) | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/mad-max-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`mad-max-vr`](https://github.com/TefMeister/mad-max-vr) |

## Shared findings

*Seeded 2026-08-26; first populated 2026-09-01 from the Mad Max dossier, so `n=1` by construction.*

- **The shader bundle ships loose, with reflection intact — and Denuvo does not cover it.**
  `[inferred-static 2026-09-01]` Anti-tamper blocks attaching to the executable, which is what
  stalled this project at first injection. It does **not** protect `Shaders_F.shader_bundle`, which
  sits in the game root carrying **1363 DXBC shaders with their `RDEF` reflection chunks intact**
  (Shader Model 5 throughout) across **84 distinct constant-buffer layouts**. Camera work is
  therefore startable with no debugger, no capture and no launch. See
  [read the shipped files before you attach anything](../techniques/#read-the-shipped-files-before-you-attach-anything).
- **The per-object camera transform is named and located:** `cbuffer InstanceConsts`, 368 bytes,
  used by 112 shaders, with **`WorldViewProjMatrix` at offset 0** (64 bytes) and `SkyMaskProjMatrix`
  at +288. Other variants carry `SpotProjectionMatrix1..3`, `SpotShadowMatrix1` and
  `PointlightProjectionMatrix1` — shadow and light passes, identified as things *not* to touch. A
  few small `$Globals` buffers hold a plain `ViewProj` / `WorldViewProj` at +0 in post and effect
  shaders.
- **⚠️ The limit, which bounds the technique for this family:** the shared per-frame buffer
  `GlobalConstants` (651 shaders, 2352 bytes) has **no recoverable member names** — its RDEF type
  record declares a raw `float4 Globals[20]` array, filled from C++, rather than a struct. So
  reflection names the per-object matrix but **cannot** name a shared view matrix. If one exists it
  must be found by value, though reflection has already narrowed that to about twenty float4 slots
  in one named buffer.


**Update 2026-09-03.** `GlobalConstants` is **two layouts, not one** `[inferred-static 2026-09-03]`:
2,352 bytes in 465 shaders (17 float4 slots plus light-position and light-colour arrays) and 512 bytes
in 186 shaders (20 slots plus a three-matrix `ShadowTransform`, very likely the shadow-pass variant).
It binds to `b0` in all 651, but `b0` is not exclusively this buffer, so a patch must key on more than
the register. The by-value probe the earlier paragraph calls for is **written** — a per-frame
constant-buffer fingerprint pass in the live-verified `dxgi.dll` proxy, compile-verified and tested
offline against constructed ground truth (17 assertions), pre-committing both readings so one launch
decides — and has **never been run against the game**. The detector design came from the sibling
`enslaved-vr` project via a `/gr` pass, which is the estate's first case of one project's validated
probe being ported to another's renderer.

**Update 2026-09-04 (from the 2026-09-03c disassembly pass).** The "very likely the shadow-pass
variant" reading above is `[disproved 2026-09-03c]`: the two `GlobalConstants` sizes are the two
**stages** — all 186 512-byte declarations are vertex shaders, all 465 2,352-byte ones pixel shaders
`[inferred-static 2026-09-03, n=651 shaders]`. Disassembling rather than reflecting settled the
camera question the probe had been designed around:

- **The shared clip transform is at vertex-side slots 0..3, written per PASS** (~10 writes per frame
  `[verified-live 2026-09-03b]`), consumed as `pos.x·M[0] + pos.y·M[1] + pos.z·M[2] + M[3]` — row-vector
  storage, translation in slot 3. Slot 4 is a per-pass view origin, slot 9 the frame-constant camera
  position that 146 vertex shaders subtract; slots 12/13 are packed fog and fade. The 4×4-shaped run at
  16..19 that the first live probe flagged is **not a matrix** (`[disproved 2026-09-03c]`: two slots
  unread by any shader).
- **Most vertex shaders do not use the shared matrix for their position.** On the chain feeding
  `SV_Position`: `InstanceConsts` b1 slots 0..3 (the per-object `WorldViewProjMatrix`) in 109 shaders,
  `cbInstanceConsts` b1 in about 35, `GlobalConstants` b0 in 15. **A VR patch on this family has two
  delivery paths to cover**, and the per-object one needs its WVP re-derived per draw — whether a
  separable world matrix exists in `InstanceConsts` slots 4..15 is the queued static question.
- The **3,136-byte** buffer the live run saw in lockstep with the 512-byte one is declared by no
  shipped shader; read as the pixel-side allocation exceeding the 2,352 bytes its shaders declare
  (ordinary D3D11 behaviour), `[hypothesis]` until the extended probe's bind census runs.

The two engine-agnostic lessons — a same-name cbuffer at two sizes is two stages, and a
constant-within-frame filter cannot see a per-pass camera — are on the techniques page
([reflection → disassembly](../techniques/README.md#reflection-gets-you-to-n-unnamed-slots-disassembly-names-them-by-use),
[per-pass filter](../techniques/README.md#-a-constant-across-every-draw-in-the-frame-filter-excludes-a-per-pass-camera-by-design)).
Tool: `dxbc-usage.py` in `flat-to-vr-RE-toolkit`, beside `dxbc-reflect.py`.

**Update 2026-09-04b — the shipped photo mode measured the projection, and the projection alone.**
Mad Max's pause-menu Capture Mode writes the same shared slots gameplay writes, which makes it a free
instrument for this family; the engine-agnostic form is
[a photo mode is also a free testbed for the camera and projection constants](../techniques/README.md#a-photo-mode-is-also-a-free-testbed-for-the-camera-and-projection-constants).

- **Its FIELD OF VIEW slider moves only the two focal columns** of the shared view-projection —
  horizontal FOV **58.3° … 116.9°**, default 80.5°, about 3° per click near the default — leaving the
  eye position, the forward column and the clip-z row untouched `[measured 2026-09-04, n=6 dumps, 5
  slider positions]`. Leaving Capture Mode with `Esc` restores the default at once, so the slider does
  not carry into gameplay by that route `[verified-live 2026-09-04, n=1]`.
- **The engine anchors the HORIZONTAL FOV and derives the vertical from the window aspect** — the same
  80.5° horizontal at 16:9 and at 1.40:1, with vertical moving from 50.9° to 62.4°
  `[measured 2026-09-04, n=2 aspects]`. A per-eye projection patch on this family must scale
  accordingly.
- **The clip-z constant is per POSITION, not per frame** — two values at two garage positions, then
  stable across thirteen dumps over ten minutes at one of them `[measured 2026-09-04, n=2 positions]`.
  Its meaning is `[hypothesis]`; what matters here is that a "constant" verified by repeated sampling at
  one spot is not thereby a frame constant. The cadence family again.
- **Practical:** the Capture Mode tabs and sliders are **mouse-only** — clicks on the tab label, the row
  label, and the `<` / `>` arrows; keys, bar clicks and knob drags do nothing `[verified-live
  2026-09-04]`. And the game's `settings.ini` stores key bindings as alphabetical indices (A=0…Z=25),
  which named the first-person-driving and enter-vehicle keys before either was pressed
  `[inferred-static 2026-09-04]`. The in-car first-person camera is still `[reported]` — pressed on
  foot with no effect, as expected, and the in-car press needs a save with a drivable car.

## See also

- [engines index](../engines-index.md) — the "Bespoke / older custom engines" row.
