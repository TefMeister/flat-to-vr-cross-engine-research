# Four library docs carry confidence tags that no tool can read — two are one character off

Filed by: `/gs`, 2026-09-01
For: `/sr` (curator of this repo)

`[verified 2026-09-01]` by the new `/gs` check 3b, which inspects the tag rather than the document.

| File | Line | Tag as written | Should be |
| --- | --- | --- | --- |
| `docs/case-studies/id-tech-6-dormant-stereo.md` | 100 | `[verified from published first-party source, 2026-09-01]` | `[reported 2026-09-01]`, source in prose |
| `docs/engines/dunia.md` | 30 | `[verified numerically 2026-09-01; not yet headset-tested]` | `[verified-numerically 2026-09-01, n=K]` |
| `docs/techniques/README.md` | 401 | `[verified across five projects, 2026-08-26 → 2026-09-01]` | `[verified-live 2026-09-01, n=5 projects]` |
| `docs/techniques/README.md` | 499 | `[verified numerically 2026-09-01, Dunia / Far Cry 2]` | `[verified-numerically 2026-09-01, n=K]` |

## The two worth pausing on

`verified numerically` is **`verified-numerically` with a space instead of a hyphen**. A human
reads those as the same tag. **No tool does.** One character decides whether the claim is counted
or invisible — and this is the strongest evidence class in the vocabulary for maths, so these are
exactly the claims you least want unreadable.

`verified-numerically` and `compile-verified` were **adopted into `CONVENTIONS.md` and the checker
today** (they were previously only in `commands/pd.md`, which is why correctly-tagged `/pd` work
was reporting as untagged). So the hyphenated form is now valid everywhere — this is a one-character
fix, not a re-tagging.

## Why this class matters

An off-vocabulary tag **reads as a strong claim to a human and counts as untagged to every
mechanical check** — the worst of both. Check 3 could never see these, because it is all-or-nothing
per *document* and these files carry plenty of valid tags elsewhere. Check 3b exists because DOOM's
dossier carried three unnoticed; the modding lane has now fixed all of its own.

## Note on `n=K`

Two of the suggested fixes want a sample count I do not have. Use what the underlying work
supports; `[verified-numerically 2026-09-01]` with the count in prose is better than a wrong `n`.
