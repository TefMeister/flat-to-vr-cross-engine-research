# SteamVR now ships a 32-bit OpenXR runtime

Supersedes: docs/runtime-layers/README.md (the OpenComposite section: "SteamVR ships no 32-bit OpenXR runtime", stated twice)

From `/gr`, 2026-10-07.

**SteamVR 2.17 left beta on 2026-09-10** with *"Added support for 32-bit OpenXR applications."*
Accepting SteamVR's "set as default OpenXR runtime" prompt registers it under both the normal and the
`WOW6432Node` `Khronos\OpenXR\1\ActiveRuntime` key `[reported 2026-10-07]`. Our dev PC shows the file
(`steamxr_win32.json`) present while the 32-bit key stayed empty until that prompt is accepted
(`alan-wake-vr` dossier, `[measured 2026-10-06]`).

Engine-agnostic: every 32-bit game routed to OpenXR (Hard Reset, Alan Wake, XIII, Far Cry 2) now has a
SteamVR option besides Virtual Desktop's VDXR, and a runtime can be picked per process with
`XR_RUNTIME_JSON` pointing at `steamxr_win32.json`. Not yet run end to end on our machines.

Sources: <https://vr.org/articles/steamvr-2-17-stable-32-bit-openxr-runtime-2026> ·
<https://www.gamingonlinux.com/2026/09/steamvr-2-17-arrives-ready-to-go-for-the-steam-frame/>
