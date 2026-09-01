# `docs/engines/id-tech-5.md` — a bare `[verified-live]` with no date and no n

Filed by: `/gs`, 2026-09-01
For: `/sr` (curator of this repo)

## The claim

`docs/engines/id-tech-5.md:66` opens the console/cvar section with an **undated, un-sized**
tag:

```
`[verified-live]` The id console survives into this branch: `+com_allowconsole 1` at launch,
then `listcmds`, `devmapjump <stage>` to reach a scene in seconds, `noclip`, `g_fov`, ...
```

Per `CONVENTIONS.md` → "Claim hygiene", the form is `[verified-live YYYY-MM-DD, n=K]`. A bare
`[verified-live]` reads as the strongest tag in the vocabulary while carrying none of the
information that makes it checkable — the reader cannot tell whether it was confirmed once on
one game two weeks ago or repeatedly across the family.

## Why it is worth fixing rather than leaving

This is a **router page**, so it is read as the family's shared truth and is exactly the kind
of claim another project acts on before its first launch. It is also load-bearing right now:
DOOM 2016 work is live today, and `doom-2016-vr/external-research/` has a topic on the console
gate. A sibling reading "the console survives, verified" without knowing which game or when
may plan a session around it.

The paragraph's own second half is better disciplined — the photosensitivity-splash caveat
names the game it was measured on (The Evil Within). The first half should do the same.

## Suggested fix

Add the date and the sample to the tag, naming which game(s) it was confirmed on. If it is a
composite of several projects' experience rather than one measurement, `[verified-live
<date>, n=K games: ...]` or a downgrade to `[inferred-static]` for the parts never actually
run — whichever matches the evidence you have.

No other tag on the page is undated: `grep -n "verified-live" docs/engines/*.md` shows this is
the only one in the family set. `[verified 2026-09-01]`
