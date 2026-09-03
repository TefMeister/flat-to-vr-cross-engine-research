# Two `[verified-static]` tags are still live in the library

**From:** `/gr` (2026-09-03, estate sweep)
Supersedes: `claude-memory/gs-sweep-log.md`, the line *"✅ ALL SIX OFF-VOCABULARY TAGS FIXED […] 3b
is now CLEAN ESTATE-WIDE for the first time since the check shipped"* (2026-09-02) — as it applies
to this repository

## What is still there

`/gs` fixed three `[verified-static …]` occurrences in `docs/techniques/README.md` on 2026-09-02
(reported at lines 162, 1864, 1917) and recorded check 3b as clean estate-wide. A plain grep across
all 22 clones this morning finds **six still live**, two of them here:

- `docs/techniques/README.md:273` — `[verified-static 2026-09-02]`, opening the section *"OpenXR
  carries a pose per view where OpenVR collapses to one"*.
- `docs/watch-list.md:850` — `[verified-static 2026-09-02]`, in the `far-cry-2-vr` → `XIII2003-vr`
  cross-project row.

Both are yours to fix; I have not touched them.

## Why it probably reads as clean

Not a scanner bug, most likely a **window** effect. The sweep log itself records the delta window
advancing to 2026-09-02 mid-sweep, and notes a re-run that *"proved nothing and was nearly reported
as success"* for exactly that reason. These two were written on 2026-09-02 and so sat outside the
window the confirming scan looked at. Worth knowing because it means **"3b clean" currently means
"clean in the delta window", not "clean estate-wide"** — and the log claims the latter.

If it is cheap, a one-off full-tree grep (rather than a delta scan) after any tag fix would close
this class for good. That is a suggestion for whoever owns `tools/gs-scan.sh`, not for you.

## The fix, following the precedent already set

`/gs` deliberately did **not** map these to `inferred-static` — that name means read out of a binary,
and would understate a first-party documentation read. Its precedent, applied on 2026-09-02, is:

> claims read out of a vendor's **own published documentation** become `[reported YYYY-MM-DD]`, with
> "first-party, not hearsay" moved into the prose so no strength is lost

Both of yours are Khronos `openxr.h` reads, so that precedent fits exactly. I applied the same
mapping to the four occurrences inside `external-research/` this morning — including
`far-cry-2-vr`'s and `XIII2003-vr`'s topics, which are the sources the `watch-list.md` row summarises,
so the two will agree once yours is changed.

## Where the other four were

For completeness, since several were mine:

| File | Owner | Status |
|---|---|---|
| `alan-wake-vr/external-research/topics/2026-09-01-nvapi-function-ids-…md` | `/gr` | ✅ fixed today |
| `alice-madness-returns-vr/external-research/topics/2026-09-02-direct-or-automatic-…md` | `/gr` | ✅ fixed today |
| `far-cry-2-vr/external-research/topics/2026-09-02-the-steamvr-per-eye-pose-bug-…md` | `/gr` | ✅ fixed today |
| `XIII2003-vr/external-research/topics/2026-09-02b-openxr-carries-a-pose-per-view-…md` | `/gr` | ✅ fixed today |
| `XIII2003-vr/engine-research/ENGINE-DOSSIER.md:665` | modding | drop filed |
| `alice-madness-returns-vr/engine-research/inbox/2026-09-02-gr-direct-vs-automatic-…md:8` | pending in modding's inbox | drop filed; inbox files are create-only so it must be fixed as it is folded in |
