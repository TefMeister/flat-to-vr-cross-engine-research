# Inbox — hand-offs to the research sweep

The game projects on this account run several Claude sessions in parallel (hands-on modding,
per-game public research, and the cross-project sweep that curates this library), and every file
is curated by exactly one of them so concurrent sessions never collide. This library's `docs/`
pages belong to the sweep.

So when a modding or game-research session notices a finding that is engine-agnostic — true
beyond its own game — it leaves a short file **here** instead of editing the docs itself, named
`YYYY-MM-DD-<mod|gr>-<short-slug>.md`: the finding, where it came from, and why it seems to
generalise.

The sweep drains this folder at the start of every run — verifies each finding, folds it into
the right `docs/` page in its own words with credit and links, then deletes the inbox file. If
this folder contains only this README, nothing is waiting.

## ⚠️ Read the whole inbox before draining any of it

Files here are **create-only**: nobody edits or deletes an existing one, not even their own from
an earlier session. That is what keeps concurrent sessions from ever colliding — but it also
means a correction cannot change the file it corrects. It arrives as a **separate, later file**
naming its target:

```
Supersedes: 2026-08-27-mod-never-dispatch-engine-commands-from-render-hooks.md
```

So before folding anything into `docs/`, run:

```
grep -r "^Supersedes:" inbox/
```

Draining oldest-first without that check writes a claim into the curated library and only then
meets the correction that withdraws it. That is not hypothetical: a 2026-08-27 finding was still
sitting here undrained when its 2026-08-28 correction arrived. Had it been curated on time, this
library would have gained a claim we now know to be false.

## Tag how well each finding is actually known

Put a confidence tag next to the claim itself — `[verified-live YYYY-MM-DD, n=K]`,
`[measured YYYY-MM-DD]`, `[inferred-static]`, `[reported]`, `[hypothesis]`, or
`[disproved YYYY-MM-DD]`. **`n=1` is not verified.** A finding that arrives untagged is treated
as `[hypothesis]`. Full definitions: `claude-memory/CONVENTIONS.md`, "Claim hygiene".
