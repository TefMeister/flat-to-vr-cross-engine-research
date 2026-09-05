# `static-disasm.py xrefs` is blind to RIP-relative references — every x64 "no xrefs" result is suspect

From: `/pd` (modding lane), home PC, 2026-09-05. Engine-agnostic, so it comes here rather than to
one game's dossier.

## The finding

`flat-to-vr-RE-toolkit/tools/static-disasm.py`'s `find_xrefs()` matches exactly two things:

1. the target address as a whole little-endian pointer (4 bytes on x86, 8 on x64), anywhere in any
   section; and
2. `E8`/`E9` rel32 call/jump displacements that resolve to the target, in executable sections.

**It never decodes ModRM, so it cannot see a RIP-relative operand** — `lea rax, [rip+disp32]`,
`mov rax, [rip+disp32]` and friends. `[verified-numerically 2026-09-05]` (read directly out of
`find_xrefs`, lines 191–216 as of this date; the tool's own docstring already says "E8/E9 rel32
calls+jumps, absolute immediates", so this is under-reading rather than a hidden bug).

## Why it matters beyond one project

On x86-32, data was addressed by absolute immediate, so the whole-pointer scan caught nearly
everything and an empty result was decent evidence of absence. **On x64 the compiler emits
RIP-relative addressing for most data references**, and the absolute pointer only survives where
something stored it in memory. So the same command that was near-conclusive on our 32-bit targets
(Alan Wake, Alice, Manhunt, XIII, PoP, Psychonauts) is close to meaningless as a negative on our
64-bit ones (DOOM 2016, Mad Max, The Evil Within, Far Cry 2, Enslaved — check each project's own
bitness before applying this).

**Any past conclusion of the form "X is not referenced anywhere, so it is dead / generic / unused"
that was drawn from `xrefs` on a 64-bit image should be re-checked before it is relied on.** I have
not audited the estate for such claims; this is a flag, not a finding about any specific one.

## What has been done already

- `static-disasm.py` now prints an explicit warning on the zero-hit path **only when the image is
  64-bit** — it stays silent for 32-bit images and whenever hits exist. Verified all three ways on
  this machine. `[verified-numerically 2026-09-05]`
- A working RIP-relative scanner exists and is committed:
  `doom-2016-vr/dev-archive/recon/2026-09-05-reflection-eye-field-hunt/tools/riprefs.py`.

## The suggestion for the library

Two things a `/sr` curator may want, neither of which I have done:

1. A line in whichever page covers static-analysis method, saying plainly that an x64 "no xrefs"
   result is not an absence proof, and naming `riprefs.py` as the thing that closes the gap.
2. Folding `riprefs.py` into the toolkit proper as a `riprefs` mode of `static-disasm.py`, so the
   right tool is the default rather than a copy living in one game's recon folder. I deliberately
   did not do this myself: it changes a shared tool's interface, and the DOOM copy is the only
   thing that has actually been exercised against a real binary so far (n=1 image).

## Provenance

Surfaced while walking DOOM 2016's reflection database, where `xrefs` reported the reflection
tables as having no code references at all. `riprefs.py` was written to check that, and confirmed
the tables genuinely have none — but only after establishing that the original tool could not have
seen them either way. The DOOM conclusion stands; the method by which it was nearly reached does
not.

**No game was launched and nothing was run against a live process.**
