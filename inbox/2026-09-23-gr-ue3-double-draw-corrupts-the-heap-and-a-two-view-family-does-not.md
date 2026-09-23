# UE3: calling the viewport Draw twice corrupts the heap; asking the engine for a two-view family does not

**From:** `/gr` estate sweep, home PC, 2026-09-23.
**For:** `docs/engines/` UE3 family page (projects: borderlands-goty-vr, bulletstorm-vr, enslaved-vr,
alice-madness-returns-vr).

Mastersellz's **BL1GOTYVR** (<https://github.com/Mastersellz/BL1GOTYVR>, `docs/HOOK_RESEARCH.md`; no
licence found, study only), a headset-tested VR mod for Borderlands GOTY Enhanced (UE3, Win64,
D3D11), reports `[reported]`:

1. **Re-entering `GameViewportClient::Draw` twice per frame corrupts the UE3 heap** (`0xC0000374`).
   Their stable path is alternate-eye: one Draw per frame, camera saved → posed → restored.
2. **Same-frame stereo without re-entry:** give UE3's own render-command constructor a temporary view
   family holding **two** principal views. The engine then does every allocation, copy-construction,
   resource registration and destruction itself; ran 3,900+ frames clean. Changing only the view
   count on the existing command is unsafe (no spare capacity; the copy constructor is non-trivial).
3. **The scene view is owned by a render-thread command** that may destroy it before returning, so
   writing or restoring its matrices after the call faults. Two restore caches inside the view also
   have to carry the pose.
4. **Camera discovery by reflection:** GNames/GObjects found by scanning and validating, camera
   fields read from `UProperty` offsets; `Default__*` objects must be rejected as live cameras.
5. **MinHook: queue all hooks and apply them atomically**; enabling individually raced the render
   thread.

Game-specific details stay in `borderlands-goty-vr/external-research/`. Found through phunkaeg's
*VR Modding Playbook* (<https://github.com/phunkaeg/vr-modding-playbook>), which the modding lane
already dropped here this morning as a general resource.
