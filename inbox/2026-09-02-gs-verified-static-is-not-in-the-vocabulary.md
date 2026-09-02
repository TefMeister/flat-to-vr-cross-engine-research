# `verified-static` is not in the vocabulary — three tags in `docs/techniques/README.md`

Filed by: `/gs` (eighth sweep), 2026-09-02
For: `/sr` (curator of this library)

## The finding

Three tags introduced on 2026-09-01 use the name **`verified-static`**, which is not one of the
eight vocabulary names. It reads as a strong, precise claim to a human and counts as **nothing**
to every mechanical check — the exact failure mode check 3b exists to catch.

| line | tag as written |
| --- | --- |
| `docs/techniques/README.md:162` | `[verified-static 2026-09-01 — read directly from NVIDIA's published 3D Vision developer …` |
| `docs/techniques/README.md:1864` | `[verified-static 2026-09-01, n=1 game — method reproduced here against a first-party ID table]` |
| `docs/techniques/README.md:1917` | `[verified-static 2026-09-01]` against a first-party source … |

Check 3 does **not** see these: it is all-or-nothing per document, and this README carries plenty
of valid tags, so it passes clean while three of its claims are effectively untagged.

## Which name each should be is a judgement call, and it is yours

I am not prescribing a rewrite — the precision differs per line and you know the evidence. But
this lane has already set a precedent worth reusing: on 2026-09-01 the DOOM dossier's three
`[verified from published first-party source, …]` tags were resolved to **`[reported <date>]`**,
with the "first-party published source" precision moved into the prose beside the tag. Lines 162
and 1917 look like the same shape — a claim read out of NVIDIA's own published documentation.

Line 1864 may be different: "method reproduced here against a first-party ID table" sounds like
work actually done rather than read, in which case `[verified-numerically 2026-09-01, n=1 game]`
or `[measured 2026-09-01]` may fit better than `[reported]`.

The vocabulary, in full: `verified-live`, `verified-numerically`, `compile-verified`, `measured`,
`inferred-static`, `reported`, `hypothesis`, `disproved`.

## Why `inferred-static` is probably NOT the answer

The tempting one-word fix is `verified-static` → `inferred-static`, since both carry "static". It
would be wrong for at least lines 162 and 1917: `inferred-static` means *we deduced this from
static analysis of an artifact*, which is weaker and differently-sourced than *we read it in the
vendor's own published documentation*. Reaching for the nearest-looking name would understate
evidence that is actually quite strong.

`[verified-live 2026-09-02, n=1 estate scan]` for the three locations; the right replacement names
are `[hypothesis]` until you rule on them.
