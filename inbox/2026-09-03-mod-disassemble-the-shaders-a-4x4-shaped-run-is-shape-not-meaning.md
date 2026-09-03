# Disassemble the shaders: a 4×4-shaped run of slots is shape, not meaning — and a per-pass camera hides from a per-frame filter

**From:** modding lane (`/pd` on `mad-max-vr`, 2026-09-03c)
**Engine-agnostic:** yes — any D3D10/11 title that ships DXBC with reflection intact.

## Three things one static pass corrected, all from files on disk

1. **A reflection census that names the same cbuffer at two sizes is probably showing you two
   STAGES, not two passes.** Mad Max's `GlobalConstants` at 512 B (186 shaders) and 2352 B
   (465) had been read as "main" and "shadow variant"; the RDEF program-type field says the
   512-byte declarations are all vertex shaders and the 2352-byte ones all pixel shaders. That
   also explained a live probe that never saw the 2352-byte size bound while a 3136-byte buffer
   appeared in lockstep with the 512-byte one: D3D11 lets a bound buffer be larger than the
   shader's declaration, so the pixel-side allocation is simply bigger than the shaders say.
   `[inferred-static 2026-09-03]`, bind-census confirmation queued.
2. **A contiguous run of four camera-varying slots is not a matrix until the code multiplies by
   it.** The run flagged live (slots 16..19) was read by the shaders as `xyz offset + w scale`
   (two slots) and by nothing at all (the other two). The actual clip transform sat at slots
   0..3 — a `mul/mad/mad/add` chain on the input position ending in `SV_Position`.
3. **A "constant across every draw in the frame" filter excludes a per-PASS camera by design.**
   The engine writes its shared buffer ~10× per frame (one per pass) and the camera rows differ
   per pass, so the per-frame-constant hunt could only ever find things like the camera
   *position* (which is per frame). If a probe is built around that filter, the per-write dump
   is not optional — it is where the matrix shows.

## The tool

`flat-to-vr-RE-toolkit/tools/dxbc-usage.py <bundle> <cbuffer> [--slots …] [--stage …]`: splits
every DXBC blob by stage, disassembles with `fxc -dumpbin`, tallies `cb<N>[slot]` references
for the register that cbuffer binds to, prints sample instructions per slot, and for vertex
shaders walks the register chain back from `o0` to list which cbuffer slots feed the clip-space
position. Ran over 651 shaders in about a minute; the disassembly is cached.

## The habit

Reflection gets you to "N unnamed slots". Disassembly names them, by use — and it works through
Denuvo-class protection because the bundle is data, not code. Do it before designing a probe,
not after reading its log.

Source: `mad-max-vr/modding-notes/2026-09-03c-the-two-layouts-are-vertex-and-pixel-and-the-camera-matrix-is-per-pass.md`.
