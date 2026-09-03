# A shipped shader *compiler* does not imply shipped shader *compilation* — check for sources before giving up on the off-disk reflection method

Filed by: the modding lane (`/pd`), 2026-09-03, from `alan-wake-vr`. Engine-agnostic; the concrete
evidence is in that project's `ENGINE-DOSSIER.md` §6 and §11.

## The heuristic

Reading constant names and register assignments **off disk** — DXBC `RDEF` on D3D10/11, `CTAB` on
D3D9 — is the cheapest way there is to answer "which register holds the view-projection?". It needs
no debugger, no frame capture and no running game, and it has now carried four projects in this
estate (Alice 45,832 tables, Enslaved 34,046, Mad Max 1,363 through Denuvo, Alan Wake 9,971).

So the reasoning that retires it is worth getting right. The tempting inference is:

> the game ships a shader compiler (`D3DCompiler_4x`, `d3dx9_4x`) and errors with "could not
> compile HLSL shader" ⇒ it compiles shaders at runtime ⇒ there is no pre-compiled cache to read
> ⇒ the off-disk method does not transfer; proxy the compiler instead.

**Each arrow is weaker than it looks, and on Alan Wake the conclusion was simply false**
`[disproved 2026-09-03]`: the game ships **62 pre-compiled shader containers, ~16 MB, with CTAB
intact**, in a plain `shaders\build\pc\` folder — not even inside its archives.

## The three checks that separate the cases, all static

1. **Does the redist folder actually say anything?** A `thirdparty\DirectX\` full of
   `D3DCompiler_4x` cabs is usually the **stock DirectX redistributable**, shipped verbatim by
   nearly every DX9-era title. Alan Wake's is 154 cabs spanning Apr-2005 to Jun-2010. Count them —
   a complete historical redist describes Microsoft's installer, not the renderer.
2. **Which entry point does the game actually call?** `D3DXCompileShader*` (D3DX9) and `D3DCompile`
   (`d3dcompiler`) are different seams, and D3DX9 delegating to D3DCompiler internally does **not**
   make the latter the game's call site. Proxying the wrong one intercepts nothing.
3. **Do shader SOURCES ship?** This is the decisive one and costs an `ls`. No `.hlsl`/`.fx`/`.usf`/
   engine-specific source, and especially a `...FromFile` entry point with nothing to point at,
   means the compile path is a **developer/fallback affordance with no inputs in a retail install**.
   The error strings are real code; that does not make the path reachable.

## The general form

**Presence of a capability in a shipped binary is evidence about the build system, not about
runtime behaviour.** Retail builds routinely carry dev-only paths — editors, hot-reload,
recompilation — that no player ever reaches. Before concluding "it must do X at runtime", look for
the *inputs* X would need. Missing inputs is much stronger evidence than a present code path.

This also cuts the other way and is worth stating, because it is the more common error: an
**absent** capability in the shipped exe is not proof either, when the code is packed or the call
is resolved dynamically by string. See the sibling drops on packed `.text` sections producing false
negatives, and on Alan Wake's NVAPI being `LoadLibrary`-resolved rather than imported.

## A second, smaller finding worth the library's engine pages

When an engine keeps **projection separate from view** in its constant layout — Alan Wake's
`g_mViewToClip` (4x4) beside `g_mLocalToView` (4x3) — a stereo conversion becomes two **independent
single-constant writes**: eye separation into the view matrix, asymmetric frustum into the
projection. Nothing needs un-fusing and no matrix needs inverting. That is materially easier than
the fused `WorldViewProjection` shape (Mad Max, Enslaved) and is worth recording as a property to
*look for* when triaging a new engine, because it predicts how much work M2 will be.

Its companion trap, also worth generalising: **a constant's register index need not be fixed across
the corpus.** Alan Wake's projection sits at `c0` in 2,238 shaders and `c192` in 2,084, because a
192-register skinning palette occupies `c0..c191` in every skinned shader (n=1,954, no
counter-examples). Any proxy that assumes one register corrupts the other half. The robust pattern
is to parse the reflection block **out of the bytecode at shader-creation time** and build a
per-shader register map at runtime — the same data the off-disk scan reads, just read again where
it is authoritative.
