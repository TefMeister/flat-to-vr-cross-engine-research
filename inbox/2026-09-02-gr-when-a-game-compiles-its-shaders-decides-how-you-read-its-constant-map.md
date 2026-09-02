# When a game compiles its shaders decides how you read its constant map — three cases, three techniques

Filed by: `/gr`, 2026-09-02
Suggested home: `docs/techniques/` — beside "read the shipped files before you attach anything", which this generalises.
Origin: three projects on this account converging on the same question from different directions.

**The recurring question:** *which register or constant carries the view-projection, and what is it called?* It is the first question every conversion asks, and the technique that answers it depends on **when the game turns shader source into bytecode** — not on the engine or the API.

| When the game compiles | What ships on disk | How to read the constant map | Seen on |
| --- | --- | --- | --- |
| **Ahead of time, source shipped** | HLSL/`.usf` **source** | just read it — the engine's own reserved-register comments are usually right there | Enslaved (UE3 ships `Common.usf`; `c0` = `ViewProjectionMatrix`) |
| **Ahead of time, source stripped** | compiled bytecode **with a reflection block** | parse it — D3D9 bytecode carries `CTAB` naming every constant and register; D3D10+ carries `RDEF` | Alice: Madness Returns (45,832 `CTAB` tables), Enslaved (34,046), Mad Max (`RDEF`) |
| **At runtime** | HLSL plus a **shader compiler redistributable** | **hook the compiler** — proxy `d3dcompiler_4x.dll` and log every source, entry point and define as it compiles | Alan Wake (ships `d3dcompiler_42`/`43` cabs; fails with *"could not process hlsl shader"* without them) `[reported 2026-09-02]` |

**The tell for the third case is in the install folder, not the binary.** A shipped shader-compiler redistributable, or a startup error naming HLSL, means the bytecode does not exist until load — so there is nothing to parse off disk, and a session that goes looking for a shader cache will correctly find nothing and draw the wrong conclusion from it. That is the negative-evidence trap this library already warns about, in a new place.

**Two things worth stating with the third case:**

1. **Runtime compilation is the *easiest* of the three to read, not the hardest.** Source names its constants in plain text, and the compile call is a single chokepoint in a DLL in the game's own directory — so one proxy on one export yields the entire corpus with names, where the compiled-cache case yields registers and the source case depends on the developer having shipped the sources at all.
2. **It is also the only one of the three that is upstream of the bytecode**, so it is the one place a per-eye term could eventually be added without patching bytecode or overriding a constant. Worth recording as a property of the case; it is a large commitment and not a first move.

**The general habit this suggests:** before planning any capture, look at what the install folder says about *when* shaders become bytecode. A `d3dcompiler_*.dll` or a shader-compiler cab in the game's own tree is as informative as the renderer's import table, and it changes which technique is even applicable.
