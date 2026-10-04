# wiz3D: an open-source way to switch on the stereo renderers games already ship (HD3D, 3D Vision)

From: `/gr`, 2026-10-04. Engine-agnostic, so it is filed here for `/sr` rather than in one game's topics.

- **What it is:** effcol's wiz3D (<https://github.com/effcol/wiz3D>, LGPL 2.1, 134 stars, pushed 2026-09-25) is a
  revival of the open-sourced iZ3D driver: a universal stereo 3D wrapper for DirectX 7-11, OpenGL, AMD HD3D and
  NVIDIA 3D Vision, with the old kernel-level hooks replaced by proxy DLLs `[reported 2026-10-04]`.
- **Why it matters for flat-to-VR:** games that shipped AMD HD3D or NVIDIA 3D Vision support already render both
  eyes themselves. wiz3D's README lists HD3D as "mostly working" (a proxy enables the game's own stereo and captures
  its quad-buffer output) and 3D Vision Direct Mode as partly working on DX11 `[reported]`. That is a cheap route to
  a correct second eye before any VR plumbing.
- **Already used by two of our projects' prior art:** farmerarmor's TombRaiderVR builds its stand-in AMD driver
  DLLs (`atidxx32`, `atiadlxy`, a vendor-ID `d3d11` proxy) from wiz3D sources (its `THIRD_PARTY_NOTICES.md`), and
  `hard-reset-vr`'s research (2026-09-29, "Hard Reset is a 3D Vision Direct-mode game") already cites it.
- **Suggested for the library:** a short technique entry, "wake the stereo the game already ships", listing the two
  vendor paths, the proxy approach, and the per-game evidence (Tomb Raider 2013 HD3D via TombRaiderVR, Hard Reset
  3D Vision Direct), with the engines-index rows that mention a vendor stereo toggle as candidates.
- Sources: `tomb-raider-2013-vr/external-research/topics/2026-09-29-tombraidervr-wakes-the-hd3d-path-with-a-fake-amd-driver.md`
  (updated 2026-10-04), `hard-reset-vr/external-research/topics/2026-09-29-hard-reset-is-a-3d-vision-direct-mode-game.md`.
