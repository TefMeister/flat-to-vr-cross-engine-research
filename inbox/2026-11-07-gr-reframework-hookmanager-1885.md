# /gr 2026-11-07: REFramework's hook fix of 11-01 was partial; #1885 completes more of it (master only)

From: `/gr` estate sweep (visceral-re8-vr, re-village-scope-vr, visceral-re2-vr check-ins). For `docs/engines/re-engine.md`
beside the #1853 paragraph and the watch list. Read online, nothing tested.

- PR #1885 by wdre9, merged 2026-11-04 (nightly 01438): the first float argument's register (xmm0/xmm1) is now saved
  and restored, so a pre-hook that CHANGES that argument is honoured when the original runs; the PR itself calls this
  the "partial fix for #1853" remainder `[reported 2026-11-07, PR text]`.
- Same PR: exceptions in pre/post callbacks are caught before they cross into the game's code; a failed hook no
  longer leaves a half-made entry; hook add/remove is serialised by a lock `[reported]`.
- 2026-11-03 (#1887): inline hooks skipped on ARM64EC code (Windows on ARM only) `[reported]`.
- `pd-upscaler`, the branch the RE2 / Village / RE4 VR builds use, is unchanged since 2026-10-11 and has none of this
  `[measured 2026-11-07, GitHub API]`.

Source: https://github.com/praydog/REFramework/pull/1885 (credit: wdre9, praydog).
