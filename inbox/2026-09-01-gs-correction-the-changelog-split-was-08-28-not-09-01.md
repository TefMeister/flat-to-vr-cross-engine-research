# Correction: the changelog split was 2026-08-28, not 2026-09-01

Supersedes: 2026-09-01-gs-sr-changelog-carve-out-points-at-the-wrong-file.md (provenance only)
Filed by: `/gs`, 2026-09-01

## What was wrong

The earlier drop attributed the `STATUS.md` -> `STATUS-CHANGELOG.md` split to commit `03daa1d`
(2026-09-01 13:08, "Split the status board into one file per project"). That is wrong.

`[verified 2026-09-01]` via `git log --diff-filter=A -- STATUS-CHANGELOG.md`:

- The changelog was split out by **`0dae26d`, 2026-08-28 16:15** ("STATUS: split changelog out
  and condense Manhunt").
- `03daa1d` on 2026-09-01 was a **different** change — splitting the status *board* into
  per-project `status/<repo>.md` files. Two separate splits, four days apart, easy to conflate.

## The finding itself stands, and is worse than reported

The four documents that still aimed `/sr` at `STATUS.md` were wrong for **four days**, not four
hours. Worse: `03daa1d` **edited `commands/sr.md`** on 2026-09-01 and still did not catch it. So
this is not "a rule that had not caught up yet" — it is a stale rule that survived a direct edit
to the very file containing it.

That strengthens the case for the mechanical check rather than weakening it. A human or model
editing a file for one reason does not re-read its unrelated claims.

## Status: already fixed

All four were corrected on 2026-09-01 (user-directed) to name `STATUS-CHANGELOG.md`, with
"**never to `STATUS.md`**" added to the two lane tables:
`commands/sr.md` (x2), `CONVENTIONS.md`, `game-mod-rules.md`.

`/sr`: nothing to action beyond noting the corrected date if you fold the earlier drop into
anything durable. Drain the two together.
