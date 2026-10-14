# REFramework VR: "Single Frame Multipass" is the rendering technique to pair with DLSS (2026-10-15, home PC, modding lane)

Engine family: RE Engine (REFramework VR, praydog; the pd-upscaler builds with PureDark's plugin for DLSS / FSR2).

REFramework's VR tab has a "Rendering Technique" choice: `Alternating/AFR`, `Two Frame Sequential`, `Single Frame
Multipass` (config key `VR_RenderingTechnique_V2`, values 0 / 1 / 2). The first two spread the two eyes over two game
frames; `Single Frame Multipass` draws both eyes in one game frame. Andyalpa's advice for RE7 VR RELOADED: use
"single pass" (= Single Frame Multipass) "for better performance and DLSS".

- RE7 with Andyalpa's RE7 VR RELOADED v1.0.3, Single Frame Multipass + `TemporalUpscaler_Enabled=true` (DLSS, quality 1):
  Tefa, 2026-10-15: "that single frame multipass works really well!" `[verified-live 2026-10-15, n=1]`.
- RE2 (Visceral) has run the same two settings since before this (`re2_fw_config.txt`: `VR_RenderingTechnique_V2=2`,
  `TemporalUpscaler_Enabled=true`) -- Tefa: "there DLSS makes a massive difference" (2026-10-09).
- Village (the scope mod) runs DLSS OFF by choice (scope sharpness), so the technique there is untested with DLSS.

Why it matters `[hypothesis]`: the temporal upscaler keeps a history per frame; with the eyes on alternate frames the
history mixes two viewpoints and ghosts, and the per-eye cost is not halved anyway on these GPUs.

Setting it before the first launch works: a `re7_fw_config.txt` holding only those keys is merged by REFramework on
start (done 2026-10-15; the rest of the file was filled in by the game).
