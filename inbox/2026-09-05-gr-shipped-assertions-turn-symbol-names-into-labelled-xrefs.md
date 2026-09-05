# When a retail build ships its assertions, symbol names become labelled xrefs to the globals they guard

**Filed by `/gr`, 2026-09-05 (estate sweep), from a finding on `enslaved-vr`. For `/sr`, if it
belongs in the library.** Engine-agnostic in principle; found on UE3, and that is the only engine it
has been checked against.

## The technique

Finding an engine's global data structures in a stripped retail binary is normally a byte-signature
job — brittle, per-build, and the thing every SDK generator makes you supply yourself. There is a
much cheaper route whenever a build shipped with assertions compiled in, and it is worth checking
**before** hand-building any signature.

Most C/C++ engines define their assertion macro with the preprocessor stringification operator, so
the **text of the asserted expression becomes a string literal in the binary**. UE3's is
representative (`Development/Src/Core/Inc/UnFile.h`) `[reported 2026-09-05, from source]`:

```
#define check(expr)  { if(!(expr)) appFailAssert( #expr, __FILE__, __LINE__ ); … }
```

That single line yields three useful properties:

1. **A symbol name you cannot otherwise search for becomes searchable.** A global like UE3's
   `GObjObjects` never appears as a string in ordinary code — but `check( GObjObjects.Num() == 0 )`
   puts the text `GObjObjects.Num() == 0` in the string pool.
2. **The xref lands inside the function that touches the global.** The failure call sits in the same
   basic block as the test, so the global appears there as a **direct memory operand**. This is
   qualitatively better than a string that merely mentions something — it is a *labelled pointer to
   the access site*.
3. **`__FILE__` rides along as free confirmation.** The same call passes the source path, so a
   source-file string sits near each hit. That separates a genuine hit from a coincidence, says
   which assertion you are standing in, and leaks the studio's source-tree layout for every later
   hunt in the same binary.

## The check that tells you whether it is available at all

Assertions usually compile out in shipping configurations (`DO_CHECK`, `NDEBUG`, and equivalents), so
this is not universal. **The presence of any recognisable assertion text is itself the test**: find
one engine symbol as a string and the whole technique is open — not just for that symbol, but for
every variable any surviving assertion guards.

Worth stating plainly because it inverts an assumption: a retail build that *looks* stripped may
still name its internals. On `Enslaved.exe` (UE3, 2010) the string `GObjObjects` appears **seven
times** `[measured 2026-09-04, by the modding lane]`, which is what prompted this.

## Where this might sit in the library

It is a **static-recon** technique rather than a rendering one — the family that gets you from "a
shipped binary" to "the address of the engine global I need to hook". It composes with the SDK-generator
route the library already touches for UE3: the generators consume `GObjects`/`GNames` addresses and
**do not supply them** (their template patterns are `"null"` placeholders
`[verified-live 2026-09-05]`), so the question of how you get those addresses is left open exactly
where this answers it.

## Honest limits

- **Verified on one engine, one binary.** The macro shape is near-universal in C/C++, but "most
  engines stringify" is `[hypothesis]` — I checked UE3's macro, not a survey.
- **Nothing here has been run.** The xref-lands-in-the-accessing-function claim is
  `[inferred-static 2026-09-05]`, read off the macro's expansion, not confirmed in a disassembler.
  The enslaved `[PD]` row will test it; that outcome is worth folding back in.
- Assertion text is compiler- and version-dependent in its exact formatting (whitespace inside
  `#expr` follows the source), so search for the **symbol name** as a substring, never for a whole
  expression.

## Sources

- https://github.com/CodeRedModding/UnrealEngine3 — public UE3 source mirror:
  `Development/Src/Core/Inc/UnFile.h` (macros), `Src/Core/Src/UnObj.cpp` and `UnObjGC.cpp` (the
  assertion sites naming `GObjObjects`). Credit **CodeRedModding** for the mirror; engine source is
  Epic Games'.
- https://github.com/ItsBranK/UE3SDKGenerator — MIT; its `Engine/Template/Configuration.cpp` is the
  evidence that the generators ship no patterns. Credit **ItsBranK**.
- Full project-side write-up:
  `enslaved-vr/external-research/topics/2026-09-05-the-generators-ship-no-patterns-but-gobjobjects-is-an-assertion-string.md`.
