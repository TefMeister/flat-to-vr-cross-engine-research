# /gs: two files in this inbox must be drained TOGETHER

**Date:** 2026-08-28 · **From:** `/gs` (GitHub Sweep) · **Type:** curation notice
**Confidence:** `[verified-live 2026-08-28]` — the supersession was detected mechanically by
`claude-memory/tools/gs-scan.sh` check 2, and both files are present in this folder.

## The situation

```
2026-08-27-mod-never-dispatch-engine-commands-from-render-hooks.md   <- claim
2026-08-28-mod-correction-xiii-gpf-was-not-the-render-path.md        <- correction
```

The second carries `Supersedes:` naming the first. **Both are still undrained**, which is the
lucky case: the false claim never reached `docs/`. Draining them in date order and acting on each
as you go would write the claim into the library and only then meet its withdrawal.

## What to do

1. Read **both** before folding either.
2. The correction wins on the *evidence*: XIII's GPF was **not** caused by dispatching from a
   render-path hook. Re-arming the engine tier crashed the game again from `ULevel::Tick` with no
   render path in the stack, so `UGameEngine::Exec` is simply not callable from that harness.
3. The *recommendation* in the 08-27 file still stands as practice — re-entering engine state
   mid-render is a real hazard. **Keep the guidance, drop XIII as its worked example**, or
   relabel XIII explicitly as a case where the render-path hypothesis was tested and failed.
4. Consider carrying the transferable method lesson instead, which is better supported than
   either: *a fix that removes the symptom and its test coverage at the same time has proved
   nothing.* The dispatch-site change stopped the crash **and** stopped the faulting path from
   ever running — two effects indistinguishable from outside.

## Why this notice exists at all

This is the situation `/gs` was built for. The check that catches it distinguishes two cases
needing opposite action: target still in the inbox (this one — drain together), versus target
already drained (**the false claim is already live in `docs/` and must be corrected there**).
Nothing else in the system would have surfaced the difference.

`/gs` is read-only and may not drain an inbox — that is this repo's curator's job. Hence a notice
rather than a fix.
