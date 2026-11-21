# From /gr (visceral-re8-vr), 2026-11-21: PureDark's AFW also exists for RE Engine, not only UEVR

**For:** `docs/runtime-layers/README.md`, section "Frame warping as a plug-in: PureDark's AFW", which names only the
UEVR fork.

**Finding** `[reported 2026-11-21]`, read through the GitHub viewer and API, nothing downloaded: PureDark's fork of
praydog's REFramework (<https://github.com/PureDark/REFramework>) carries the same Alternate Frame Warping as
pre-release builds for **RE9** (`RE9_AFW_v1.0-beta.1` to `beta.6`, 2026-03-29 to 2026-07-09) and **Pragmata**
(`PRAGMATA_AFW_v1.0-beta.1`, 2026-05-19), plus branches for the **RE4 remake** (`RE4RAFW`, `RE4AFW_CPPVRMod`, March to
August 2026). The same shape as on UEVR: a compiled add-on behind a published header (`PDAFWPlugin.h`), DLSS or DLAA
must be on, about 500 MB more video memory, a volumetrics difference between the eyes, debug keys on the numpad. The
RE4 branch's commit messages say the volumetrics were fixed by putting the old view-projection matrix back before the
post-process layer. No build for Village (RE8) or the RE2 / RE3 remakes was found.

**Suggested change:** one or two sentences in that section: AFW is not UEVR-specific; the same add-on is wired into
REFramework per game (list above), so for RE Engine games it arrives as a per-game framework build, with the same
permission caveat.

**Detail:** `visceral-re8-vr/external-research/topics/2026-11-21-puredarks-afw-exists-for-re-engine-but-not-for-village.md`.
