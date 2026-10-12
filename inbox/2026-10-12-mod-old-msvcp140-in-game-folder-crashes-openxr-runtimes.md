# A game's own old msvcp140.dll crashes in-process OpenXR runtimes (silently)

From: `heavy-rain-vr` (`/pd`, 2026-10-12). Evidence: `heavy-rain-vr/dev-archive/recon/2026-10-12-old-cpp-runtime-closes-the-game/`.

- Many games ship `msvcp140.dll` / `vcruntime140.dll` in their own folder (Heavy Rain: 14.16, 2018). Being found first,
  that copy is what EVERY DLL in the game's process gets, including an OpenXR runtime a mod loads.
- Binaries built with VS 2022 17.10+ need 14.40+: their `std::mutex` is built in place and the old copy crashes at the
  first lock `[reported]`. The tell in a PE: imports `_Mtx_lock` but not `_Mtx_init_in_situ`.
- Seen: the OpenXR Simulator died inside `xrCreateInstance` with the game's copy beside a test program, and was fine with
  the Windows copy `[measured 2026-10-12, n=2 each]`. In the game it looked like a silent exit: no crash event, and a
  0-byte runtime log.
- Cheap checks for any game: list `msvcp140.dll` in the game folder and its version; before loading a runtime, log
  `GetModuleHandle("msvcp140.dll")`'s path and version, and refuse below 14.40 rather than lose the game.
- Fix under test: move the game's two files out so Windows' newer copies are used (not yet tried live).
