# `/sr`'s one shared-file carve-out now names a file it should not touch

Filed by: `/gs`, 2026-09-01
For: `/sr` (this repo's curator — but the fix lives in `claude-memory`, which has no inbox,
so this is the only place a note reaches you. Please raise it with the user / modding lane.)

## The drift

`claude-memory` commit `03daa1d` (2026-09-01 13:08, "Split the status board into one file per
project") moved the changelog **out of `STATUS.md` into its own `STATUS-CHANGELOG.md`.**

Four documents were not updated and still aim `/sr` at `STATUS.md`:

| Document | Line | Says |
| --- | --- | --- |
| `~/.claude/commands/sr.md` (and its canonical twin `claude-memory/commands/sr.md`) | 34 | "an **append-only** dated changelog line in `STATUS.md`" |
| same | 146 | "append a short dated line to `STATUS.md`'s changelog" |
| `claude-memory/CONVENTIONS.md` | 84 | "`/sr` appends one dated changelog line to `STATUS.md`" |
| `~/.claude/game-mod-rules.md` | 117 | same wording |

`[verified 2026-09-01]` — read directly from the four files and from `git log -- STATUS.md`.

## Why this one matters more than it looks

`STATUS.md` is the **most contended file in the estate**: the modding lane rewrites its
open-actions block constantly (12 separate commits touched it today between 10:15 and 13:25).
The changelog split is precisely what removed `/sr` from that contention — the whole point of
the carve-out is that appending to a *separate* file cannot conflict with a lane that is
rewriting a *shared* one.

So the written rule now points the estate's **only** cross-lane shared-file exception straight
back at the file the split was designed to get it out of. A future `/sr` that follows its
command file literally re-creates the collision the split removed.

## It has not bitten yet

Today's two `/sr` commits went to the right place — `8355c01` and `512b2fd` both touch
`STATUS-CHANGELOG.md` only. `[verified 2026-09-01, n=2]` The session evidently read the actual
files rather than the rule. That is luck, not protocol: the next run has no reason to look.

## Suggested fix (modding lane owns all four files)

Replace `STATUS.md` with `STATUS-CHANGELOG.md` in the four places above. One-word change each.
