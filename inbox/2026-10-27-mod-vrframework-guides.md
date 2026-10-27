# Lead: elliotttate's vrframework guides (AFR, frame timing, TAA history under AFR)

Source: https://github.com/elliotttate/vrframework (MIT, derived from praydog/REFramework), seen 2026-10-27.
A 17-part written guide (~6,000 lines, ~180 file:line citations) on retrofitting stereo VR into a closed engine,
comparing REFramework (RE Engine) with mutars' starfield2vr (Creation Engine 2) and anvilengine2vr (Anvil).
The code half is a scaffold with stub bodies (`PORT FROM:` pointers), not a runtime. `[reported]`

Worth folding into the library:
- guides/07 frame timing: three frame counters; AFR parity must stay locked to the eye the compositor expects;
  Reflex-marker pacing (Creation 2) versus two function hooks (Anvil); drift detection and skip-present recovery.
  Relevant to every "jittery" rate fault and to Alice: Madness Returns, which draws one eye per frame. `[reported]`
- guides/08 section 6.2: any resource or matrix carried from frame to frame (TAA colour/depth history, motion
  vectors, previous view-projection) is poisoned by AFR; Creation 2 keeps per-parity copies of the history, Anvil
  ping-pongs the previous view matrix. `[reported]`
- guides/08 section 5: before writing a matrix, hook the projection read-only and log the mono matrix first.

Got on the dev PC through Lanes 0.92.0 (`outside.py get vrframework`). Not yet applied to any of our games.
