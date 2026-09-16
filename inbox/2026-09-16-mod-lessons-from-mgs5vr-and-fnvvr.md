# Lessons from two other AI-assisted VR mods: MGS5VR and fnvvr

**From:** an ordinary session on the home PC (`RTX`), 2026-09-16, reviewing two public repositories
by the GitHub user **nikamigaming-create** (astr0h on Discord), at the user's request.
**For:** `/sr`, to curate into `docs/techniques/` and `docs/engines/` as it sees fit.

Sources (shallow clones read on 2026-09-16):
- https://github.com/nikamigaming-create/MGS5VR — **MIT licence.** Native OpenXR mod for Metal Gear Solid V (D3D11, Fox Engine). About 10.5k lines of C++ over ~40 files.
- https://github.com/nikamigaming-create/fnvvr — **no licence file, so all rights reserved.** Fallout New Vegas VR (32-bit D3D9, Gamebryo). ~130k lines. By its own capability matrix it has **never been tested on a physical headset**.

Policy reminder: ideas only, no code copied, credit both if any of this is used.

Tag meaning here: `[inferred-static 2026-09-16]` = we read the code that does it; `[reported 2026-09-16]` = only their docs say so. Neither project's results were run by us.

## Techniques

1. **Same-frame stereo by re-calling the engine's own scene draw** (MGS5VR `src/render_camera.cpp:316-520`) `[inferred-static 2026-09-16]`. Hook the scene-draw function. Inside one call: write the left-eye view into the camera object, re-run the engine's own viewport and projection builders, draw, then do the same for the right eye, then restore the originals through a scope guard. One game update and one Present give two real eye images. fnvvr plans the same for Gamebryo, with **one shared cull over a combined frustum** wide enough for both eyes, then per-eye accumulators `[reported 2026-09-16]`. Both list replaying draws or the whole frame twice as things to avoid.
2. **Previous-frame matrices get overwritten per eye** (MGS5VR) `[inferred-static 2026-09-16]`. Before each eye's draw, the "previous view" matrix is set to that eye's current matrix, so motion blur and TAA see zero motion instead of the other eye's view. A cheap fix for the smear that alternating-eye rendering gives temporal effects.
3. **Render each eye with a symmetric FOV that encloses the headset's asymmetric one, then crop at submission** (MGS5VR `src/stereo.cpp:96-121`, `xr_runtime.cpp:1029-1044`, `enclosingEyeFov` / `eyeImageRegion`) `[inferred-static 2026-09-16]`. The crop is `XrSwapchainSubImage.imageRect`, worked out with tangent maths. They report this fixed sky and lighting glitches that off-centre projections caused in the engine `[reported 2026-09-16]`. Relevant anywhere a game's sky or post-processing assumes a centred projection.
4. **Tell a real view matrix from an impostor: world × view must be within 0.003 of identity** (MGS5VR `render_camera.cpp:119-140`) `[inferred-static 2026-09-16]`. If the check fails, head tracking is cancelled for that frame.
5. **Fail closed on unexpected code bytes.** MGS5VR compares the live bytes at every hook target with an expected pattern and checks the exe's SHA-256. fnvvr hashes every hooked function body before patching. Both refuse to install on a mismatch `[inferred-static 2026-09-16]`. fnvvr reports catching another mod (JIP LN) that had already patched a function `[reported 2026-09-16]`. Useful for multi-version games, and for noticing when Denuvo or a patch changed the code.
6. **Prove which shader constants hold the camera instead of guessing** (fnvvr `scripts/get-verified-shader-wvp-contracts.ps1`) `[inferred-static 2026-09-16]`. Disassemble each vertex shader with `fxc`, trace data flow from the constant register to `oPos`, and reject shaders that branch. Only shaders that pass go on an allow-list and get patched. The same allow-list idea applies to D3D11 shaders, which 3Dmigoto can dump. fnvvr's correction is D = E·C⁻¹ in double precision, with a residual check (`renderhook/fnvxr_d3d9_proxy.cpp:3977`) `[inferred-static 2026-09-16]`.
7. **Bridge a 32-bit game to a 64-bit OpenXR runtime with a separate host process** (fnvvr) `[inferred-static 2026-09-16]`. The link is named shared memory with a magic number and a version in its name. **Every struct size and field offset is checked at compile time for both 32- and 64-bit builds** (`protocol/fnvxr_gpu_color_transport.h:104-118`) and again in a runtime test. The frame buffer has several slots with separate reader lanes, so a crashed reader cannot stall the game. The GPU route (D3D9Ex → NT-handle shared D3D11 texture + `ID3D11Fence`) is **not proven live** by their own account `[reported 2026-09-16]`.
8. **Controller input without SendInput or window focus** (fnvvr) `[inferred-static 2026-09-16]`. Fake XInput and DInput proxies read controller state from shared memory, with `dwPacketNumber` taken from that state.
9. **The OpenXR frame loop never blocks the game** (MGS5VR) `[inferred-static 2026-09-16]` for the loop, `[reported 2026-09-16]` for the threading. The loop runs on its own worker thread, and a scope guard always calls `xrEndFrame`. The game drops eye frames into a keyed-mutex "mailbox" texture. If the game stalls, the last pair is shown again for up to 500 ms, with its original poses. The pair is rejected if the pose is over 150 ms old (fnvvr: 25 ms).
10. **HUD handling** `[reported 2026-09-16]`. MGS5VR routes the engine's own UI jobs, carrying each eye's view and projection, onto a panel on the forearm; the crosshair and waypoint letters are hidden, not head-locked; menus switch to a big virtual screen. fnvvr: gameplay has no HUD, and every blocking menu becomes a mono floating panel. On leaving a menu, the panel stays until a fresh stereo frame arrives, and it shows black rather than a stale frame.
11. **Test without the headset: Meta XR Simulator plus scripted motion** (MGS5VR `tools/launch-simulator.ps1`, `tools/simulator-motion.py`, `tools/record-simulator.py`) `[inferred-static 2026-09-16]`. A per-process OpenXR runtime override points the game at the simulator, which runs headless at 1280×720. Head and controller motion is scripted with smooth curves through OpenXR actions, and ~15 s single-eye clips are recorded as evidence. Their contributing guide bans Windows mouse and keyboard automation. **Directly relevant to our dev PC, which has no headset.** Not tried by us.
12. **Keep a launcher's debugger away from XR runtime helpers.** MGS5VR calls `NtSetInformationProcess` with class 31 so Ground Zeroes' launcher-as-debugger does not adopt the runtime's helper processes (`src/proxy.cpp:36-66`) `[inferred-static 2026-09-16]`.

## Engineering practices (also filed for the estate in `claude-memory`)

- Pure maths and logic in a library with no game or D3D dependency, tested on its own; game-facing code in a second library (MGS5VR `CMakeLists.txt:19-28`).
- Engine addresses and offsets in one layout struct per game build, plus a per-build JSON evidence profile with `verified` flags. Their runtime does **not** read that JSON `[inferred-static 2026-09-16]`.
- fnvvr's **capability matrix**: columns Implemented / Unit-tested / Simulator-tested / Physically-tested.
- fnvvr's **mutation-site inventory**: every patch, what it validates against, and what must be restored.
- MGS5VR's **acceptance table** with a *negative fixture* column (what must NOT happen).
- fnvvr's **"fuse" tests**: CMake scripts that fail the build if a safety guard disappears from the source.
- Counter-examples: fnvvr has 13k–21k-line files and ~605 environment-variable settings instead of a config file. MGS5VR still keeps timeouts and thresholds as literals in code.

## Why this generalises

Items 2–6 and 9–11 depend on no particular engine. Items 1 and 7 need engine function addresses (1) or a 32-bit game (7), and those describe several of our projects (Psychonauts, Alan Wake, Manhunt, Burnout Paradise, Prototype).
