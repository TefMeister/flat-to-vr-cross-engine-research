# Xbox 360 LithTech Jupiter EX (Condemned 2): world unit from the game database, and an off-centre view for free

From: `/lm condemned-2-vr`, 2026-10-15. Detail: `condemned-2-vr/engine-research/ENGINE-DOSSIER.md` §8a.

- **World unit = 1 cm** `[inferred-static 2026-10-15]`, read from `database\Yeti.gamedb` inside
  `Layer01.Arch02`: player box half-sizes (40, 95, 40), walk 235/s, gravity -2000/s², jump 800/s; only
  centimetres fit every value. Method worth reusing on other Xbox 360 Jupiter EX titles (F.E.A.R. 2,
  F.E.A.R. Files) `[hypothesis]`: an `.Arch02` reader, the XMem LZX decompressor (libmspack), and the
  gamedb name hash `h = h*919 + code(c)` (a-z/A-Z 1..26, 0-9 27..36, `_` 38) `[verified-numerically 2026-10-15, n=9]`.
  Tools: private `staging/condemned-2-vr/world-unit-2026-10-15/`.
- **The engine's view setup already supports an off-centre projection:** set view takes a second float
  pair after the FOV tangents that is the picture centre in tangent units (`ndc = (tan - centre) /
  half-width`). So a headset's uneven per-eye view fits exactly with half-widths (L+R)/2, (U+D)/2 and
  centre (R-L)/2, (U-D)/2, no matrix patching needed `[verified-live 2026-10-15, n=1]` (OpenXR simulator).
  General lesson: before patching projection matrices, look for an existing "offset"/"shift" input on
  the engine's view call; older engines often kept one for split screens or jitter.
