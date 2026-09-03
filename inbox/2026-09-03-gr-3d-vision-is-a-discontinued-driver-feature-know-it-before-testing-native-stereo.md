# 3D Vision is a discontinued driver feature (last driver 425.31, April 2019) — know it before any project tests a game's native stereo toggle

Filed by: `/gr`, 2026-09-03
Origin: `alice-madness-returns-vr/external-research/topics/2026-09-03b-3d-vision-automatic-on-a-current-driver-what-it-takes-and-why-it-is-not-a-vr-route.md` (pointer row also in `alan-wake-vr`)
Suggested home: `docs/generic-drivers/` or `docs/landscape/` — wherever the library already discusses 3D Vision / Helix / geo-11 (a grep finds those names on nine pages, but **not the driver cut-off versions**, which is the gap this fills)

Why it generalises: at least two projects on the account (Alice: UE3/D3D9; Alan Wake: Remedy/D3D9) shipped **native NVIDIA 3D Vision Automatic** support, both now confirmed Automatic by the same static caller-count method, and any other 3D-Vision-era title will hit the same wall the moment someone ticks its stereo option to "see what the developers' stereo looks like".

- NVIDIA announced the end of 3D Vision driver support on **2019-04-11**; **425.31 is the last driver that includes it**. `[reported]` (Wikipedia, citing NVIDIA's support-plan notice — NVIDIA's own page is 403 to automated fetch)
- DX11 stereo lingered to **452.06** through community workarounds and was removed with the RTX 30-series launch driver (Oct 2020). `[reported]` (3D Fix Manager, Pauldusler; HelixVision notes, Bo3b)
- **DX9 games are *reported* to remain 3D-Vision-compatible on current drivers** — one source (3D Fix Manager), unconfirmed anywhere in the estate. The most consequential line for the D3D9 projects, and the one to verify if any of them ever wants a reference picture.
- The driver-made stereo also needs a display: 120 Hz 3D Vision monitor + emitter, 3DTV Play, or **Discover-mode anaglyph glasses on any GeForce**. `[reported]`
- **geo-11** (davegl1234, June 2022) is a full replacement stereo driver — **DX11 only**; DX9 titles reach it through dgVoodoo2 translation, about half successfully. `[reported]` (Helix Mod)
- ⚠️ Framing the library should keep: **none of this is a VR route.** 3D Vision drives a display, not a headset; its value to a flat-to-VR project is the *shader plumbing it left behind* (`nvstereo.h` `StereoParmsTexture` consumers, driver-published separation/convergence), which a proxy can feed itself.

Suggested library change: one short "3D Vision is dead — what that means for native-stereo toggles" note carrying the three version numbers (425.31 / 452.06 / 456.38) and the DX9 caveat, linked from the pages that already mention Helix/geo-11, so no project spends a flat run discovering that a stereo checkbox does nothing.
