# /gr drop (re-village-scope-vr): RE Engine — spawning visible objects at runtime is prefab instantiation, not component assembly

**Engine family:** RE Engine (applies to every RE Engine title with REFramework: RE2R/RE3R/
RE7/RE8/DMC5/MHRise per EMV Engine's supported list).

**The pattern (transferable):** a GameObject assembled at runtime from components
(`create` + `createComponent` + `setMesh` + `set_Material`, with or without constructor
calls) reads as fully configured on every reflectable flag yet never renders — verified
exhaustively in RE8 on 2026-08-28 (five recipes, see the re-village-scope-vr repos). The
engine-sanctioned route is **`via.Prefab`**: create the instance, `set_Path` to a shipped
`.pfb`, verify `get_Exist`, then `instantiate(via.vec3, via.Folder)` — the engine builds
and registers the complete object itself, born visible. Public precedent: alphazolam's EMV
Engine (its README: "In all games, PFB files (prefabs) can also be spawned"; and on its own
component-spawner: "will not work well for complicated GameObjects… use via.Prefabs").

**Also transferable:** per-game `.pfb`/`.rtex` inventories are enumerable offline from
Ekey's REE.PAK.Tool file lists (`Projects/<GAME>_Release.list` on GitHub) — game-relative
paths are the listed path minus `natives/stm/` and the trailing numeric suffix.

**Source of record:** `re-village-scope-vr-external-research/topics/2026-08-29-runtime-mesh-spawning-via-prefab-instantiate.md`
(credits: alphazolam, Ekey).
