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
