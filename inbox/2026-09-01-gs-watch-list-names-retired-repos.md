# `docs/watch-list.md` still names pre-consolidation repos in five places

Filed by: `/gs`, 2026-09-01
For: `/sr` (curator of this repo)

The 2026-08-30 consolidation turned `<prefix>-<lane>` repos into folders inside one repo per
game. `docs/watch-list.md` still uses the old repo names:

| Line | Names | Now |
| --- | --- | --- |
| 120 | `doom-2016-vr-engine-research` | `doom-2016-vr/engine-research/` |
| 125 | `manhunt-2003-vr-engine-research` | `manhunt-2003-vr/engine-research/` |
| 129 | `doom-2016-vr-external-research`, `manhunt-2003-vr-external-research` | `<prefix>/external-research/` |
| 183 | `XIII2003-vr-external-research/inbox/...` | `XIII2003-vr/external-research/inbox/...` |
| 184 | `doom-2016-vr-engine-research/inbox/...` | `doom-2016-vr/engine-research/inbox/...` |

`[verified 2026-09-01]` — grepped and read from the file.

## Why it is worth a pass rather than leaving to rot naturally

The old repos **still exist as frozen duplicates** until the user-approved deletion pass
(`claude-memory/consolidation-2026-08-30.md`), so these names still resolve. A reader following
one lands in real-looking but stale content, with no error to warn them — and the standing rule
is *never push to those*. A dead link would be safer than a live wrong one.

This file is also `/sr`'s own coverage bookmark, so the names get re-read every sweep.

## Scope note — most of the estate's stale names are NOT worth fixing

The same grep across all 22 repos hits mostly **dated historical entries**
(`modding-notes/2026-08-25-*.md`, `dev-archive/notes/*`). Those correctly name the repo that
existed on the day they were written, and rewriting them would falsify the record. The ones
worth fixing are live forward-looking docs like this one. Reported to the user in full; drops
filed only where a reader would be actively misled.
