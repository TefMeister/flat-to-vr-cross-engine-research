# /gr 2026-11-11: the OpenXR Simulator's master lifts the 4096 picture cap for Direct3D (unreleased)

From: `/gr` on silent-hill-2-remake-vr. For `docs/runtime-layers/README.md` → "Headset-free testing: OpenXR-Simulator
and its forks".

**Finding.** elliotttate/OpenXR-Simulator commit 692a66d (2026-10-29, on `master`, after the v1.7.1 release of
2026-10-28): the runtime used to advertise and enforce a 4096 swapchain limit for every graphics binding; Direct3D 11
and 12 instances now get 16384 (the feature-level 11.0 texture limit) in the system properties, the view
configuration's largest image size and the `xrCreateSwapchain` check alike; OpenGL, Vulkan and mixed bindings keep
4096 `[measured 2026-11-11, commit message]`. Silent Hill 2 found the old cap the hard way: at the headset's own size
(5376 wide) UEVR never started VR and the game ran flat with no warning (SH2 board, 2026-11-10). No release carries the
change yet `[measured 2026-11-11, releases list]`.

**Suggested library line.** "Simulator v1.7.1 and older cap every picture at 4096 wide: a game asking for more fails
to start VR, sometimes silently. Measure below 4096 per eye and scale the load with the engine's screen percentage, or
use a build from `master` after 2026-10-29 (Direct3D only)."

Related, already in the SH2 topic and not repeated here: since v1.6.0 the simulator aborts at process exit unless
`xrDestroyInstance` ran (its preview thread), see
https://github.com/TefMeister/silent-hill-2-remake-vr/blob/main/external-research/topics/2026-11-11-the-quit-abort-is-the-simulators-preview-thread.md

Source: https://github.com/elliotttate/OpenXR-Simulator/commit/692a66d (credit: elliotttate).
