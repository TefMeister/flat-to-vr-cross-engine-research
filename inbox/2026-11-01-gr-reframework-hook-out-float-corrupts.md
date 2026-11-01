# REFramework: hooking a method with an out/ref float argument corrupts memory (fixed on master 2026-11-01)

From: `/gr` estate sweep, 2026-11-01.

- praydog/REFramework #1853: HookManager picks an XMM register for any `System.Single` argument, but an `out`/`ref
  System.Single` is a pointer passed in RDX/R8/R9; the hook restores XMM and not the integer register, so the original
  method then gets a bad pointer `[reported]`. Fixed on **master** by `bda3f6d` "HookManager: Use integer registers for
  out/ref float args (fixes #1853)" and `1813bf3` "Handle by-value double args and returns in XMM", both 2026-11-01;
  in REFramework nightly 01435 / 01436 `[reported]`.
- **The `pd-upscaler` branch has no commits since before 2026-10-20**, so builds from it (our RE2 and Village builds,
  Talemann's RE4 VR) do not have the fix `[measured 2026-11-01, GitHub commit list]`.
- Rule for any RE Engine mod on those builds: do not `sdk.hook` / `add_hook` a method whose parameters include an
  `out` or `ref` float; hook a neighbour, or read the value after the call.
- Library page: docs/engines/re-engine.md (tooling cautions).
- Sources: <https://github.com/praydog/REFramework/issues/1853>,
  <https://github.com/praydog/REFramework/commit/bda3f6d>, <https://github.com/praydog/REFramework/commit/1813bf3>.
