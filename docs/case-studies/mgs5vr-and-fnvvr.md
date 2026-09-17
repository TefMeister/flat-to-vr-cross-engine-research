# Two AI-assisted native VR mods: MGS5VR and fnvvr

*Study notes on two public projects by GitHub user **nikamigaming-create** (astr0h on Discord). Read on
2026-09-16 by one of this account's sessions, at the account owner's request, and curated here on
2026-09-17. **Ideas only, in our own words — no code copied.** Credited in
[`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).*

| Project | Game / engine | Licence | Status by its own account |
| --- | --- | --- | --- |
| [MGS5VR](https://github.com/nikamigaming-create/MGS5VR) | Metal Gear Solid V (Fox Engine, D3D11), native OpenXR | **MIT** | About 10.5k lines of C++ |
| [fnvvr](https://github.com/nikamigaming-create/fnvvr) | Fallout: New Vegas (Gamebryo, 32-bit D3D9) | **none — all rights reserved** | About 130k lines; its capability matrix says it has **never been tested on a physical headset** |

**Tags:** `[inferred-static 2026-09-16]` means our session read the code that does it;
`[reported 2026-09-16]` means only the project's own documentation says so. **We ran neither.**

## Rendering

1. **Same-frame stereo by re-calling the engine's own scene draw** (MGS5VR) `[inferred-static]`. Hook
   the scene-draw function. Inside one call: write the left eye's view into the camera object, re-run
   the engine's own viewport and projection builders, draw; repeat for the right eye; restore the
   originals through a scope guard. One game update and one `Present` give two real eye images. fnvvr
   plans the same on Gamebryo with **one shared cull over a combined frustum** wide enough for both eyes
   `[reported]`. Both list "replay the draws" and "render the whole frame twice" as things to avoid.
   Compare [synchronized sequential](../techniques/README.md#stereo-submission-strategies).
2. **Overwrite the previous-frame matrices per eye** (MGS5VR) `[inferred-static]`. Before each eye,
   set the "previous view" matrix to that eye's current one, so motion blur and TAA see zero motion
   rather than the other eye's view. A cheap fix for
   [temporal effects under per-eye rendering](../techniques/README.md#temporal-effects-under-afr).
3. **Render a symmetric FOV that encloses the headset's asymmetric one, then crop at submission**
   (MGS5VR) `[inferred-static]`. The crop goes into `XrSwapchainSubImage.imageRect`, worked out from the
   eye tangents. The project reports it fixed sky and lighting glitches caused by off-centre projections
   `[reported]`. Worth trying anywhere a game's sky or post-processing assumes a centred projection.
4. **Tell a real view matrix from an impostor: world × view within 0.003 of identity** (MGS5VR)
   `[inferred-static]`; head tracking is cancelled for any frame that fails. A concrete instance of
   [main-camera discrimination](../techniques/README.md#main-camera-discrimination).
5. **Prove which shader constants hold the camera instead of guessing** (fnvvr) `[inferred-static]`.
   Disassemble every vertex shader, trace data flow from the constant register to the output position,
   reject shaders that branch, and patch only an allow-list of shaders that pass. The correction is
   computed in double precision with a residual check. The same allow-list idea applies to D3D11
   shaders dumped with 3Dmigoto.

## Runtime and process structure

6. **Fail closed on unexpected code bytes** (both) `[inferred-static]`. MGS5VR compares the live bytes
   at every hook target with an expected pattern and checks the executable's SHA-256; fnvvr hashes every
   hooked function body. Both refuse to install on a mismatch. fnvvr reports catching another mod that
   had already patched a function `[reported]`. Useful on multi-version games, and for noticing a patch
   or protection layer changing the code.
7. **A 32-bit game can reach a 64-bit OpenXR runtime through a host process** (fnvvr)
   `[inferred-static]`. Named shared memory carries a magic number and a version in its name; **every
   struct size and field offset is checked at compile time in both 32- and 64-bit builds**, and again in
   a runtime test. Several frame slots with separate reader lanes mean a crashed reader cannot stall the
   game. Its GPU route (D3D9Ex → NT-handle shared D3D11 texture + `ID3D11Fence`) is **not proven live**
   by its own account `[reported]`. Compare the
   [D3D9 shared-handle bridge](../techniques/README.md#d3d9-to-a-modern-vr-compositor-the-shared-handle-bridge-and-its-two-traps).
8. **Controller input without `SendInput` or window focus** (fnvvr) `[inferred-static]`: fake XInput
   and DirectInput proxies read controller state from shared memory, with `dwPacketNumber` taken from
   that state.
9. **The OpenXR frame loop never blocks the game** (MGS5VR) — loop `[inferred-static]`, threading
   `[reported]`. The loop runs on its own worker thread and a scope guard always calls `xrEndFrame`. The
   game drops eye frames into a keyed-mutex "mailbox" texture; if the game stalls, the last pair is shown
   again for up to 500 ms with its original poses, and a pair whose pose is more than 150 ms old is
   rejected (fnvvr: 25 ms).
10. **Keep a launcher-as-debugger away from XR runtime helpers** (MGS5VR) `[inferred-static]`: it calls
    `NtSetInformationProcess` (information class 31) so the game's launcher does not adopt the runtime's
    helper processes.

## HUD

11. `[reported]` MGS5VR routes the engine's own UI jobs, with each eye's view and projection, onto a
    forearm panel; crosshair and waypoint letters are hidden rather than head-locked; menus become a
    large virtual screen. fnvvr has no gameplay HUD, and every blocking menu becomes a mono floating
    panel that stays until a fresh stereo frame arrives, showing black rather than a stale frame. See
    [HUD & UI in VR](../techniques/README.md#hud--ui-in-vr).

## Testing without a headset

12. **Meta XR Simulator plus scripted motion** (MGS5VR) `[inferred-static]`. A per-process OpenXR
    runtime override points the game at the simulator, which runs headless at 1280×720. Head and
    controller motion is scripted as smooth curves through OpenXR actions, and short single-eye clips are
    recorded as evidence. Its contributing guide bans Windows mouse and keyboard automation. **Directly
    relevant to any development machine without a headset. Not tried by us.**

## Engineering practices worth copying (as ideas)

- Pure maths in a library with no game or D3D dependency, tested on its own; game-facing code in a
  second library (MGS5VR).
- Engine addresses and offsets in one layout struct per game build, with a per-build evidence profile
  carrying `verified` flags (MGS5VR).
- A **capability matrix** with columns Implemented / Unit-tested / Simulator-tested / Physically-tested
  (fnvvr) — an honest status table that cannot quietly promote "built" to "works".
- A **mutation-site inventory**: every patch, what it validates against, and what must be restored
  (fnvvr).
- An acceptance table with a **negative fixture** column — what must *not* happen (MGS5VR).
- **"Fuse" tests** that fail the build if a safety guard disappears from the source (fnvvr).
- Counter-examples, also instructive: 13k–21k-line source files and ~605 environment-variable settings
  in place of a config file (fnvvr); timeouts and thresholds left as bare literals (MGS5VR).

## What generalises

Items 2–6 and 9–12 depend on no particular engine. Item 1 needs the engine's scene-draw and camera
builders; item 7 applies to any 32-bit game, which on this account includes Psychonauts, Alan Wake,
Manhunt, Burnout Paradise and Prototype.
