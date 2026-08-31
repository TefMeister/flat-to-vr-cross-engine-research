# Build trap: `core.autocrlf` makes a self-verifying build look catastrophically broken

**From:** modding session (`doom-2016-vr`), 2026-08-31
**Kind:** tooling trap, engine-agnostic, affects any project in the estate
**Confidence:** `[verified-live 2026-08-31, n=1]` — hit and fixed today

## What happened

`doom-2016-vr`'s Vulkan proxy has a genuinely good safety check: the build **fails** unless every
one of the 96 functions the game imports is present in the built DLL. It compares a checked-in
list (`gen/doom-imports.txt`) against `llvm-objdump` output using `comm`.

After re-cloning the repo on a Windows machine, that check reported **all 96 imports missing**.
The build looked completely broken. Nothing was wrong with it.

**Cause:** git on Windows was configured with `core.autocrlf=true`, so the checked-in list arrived
with **CRLF** line endings while the tool output had LF. `comm` was comparing
`vkAcquireNextImageKHR\r` against `vkAcquireNextImageKHR`. Every line failed to match, which is
indistinguishable from every symbol genuinely being absent.

The neighbouring check — a `wc -l` **count** comparison — passed happily, because counting lines
does not care about their endings. So one half of the verification said "fine" and the other said
"total failure", which made it read like a real and severe regression.

## Why it is worth a library page

This is latent in **any** repo whose build verifies itself by comparing a checked-in text list
against tool output — export lists, symbol lists, expected-file manifests, golden outputs. It is
invisible until someone clones fresh on a machine with `autocrlf` on, and at that moment it
produces the most alarming possible false negative.

It also has a nasty second-order effect: the natural reaction is to distrust the code you just
wrote. Today that cost a detour before the line endings were checked.

## The fix, and why it goes in the script

```sh
comm -23 <(tr -d '\r' < gen/doom-imports.txt | sort -u) \
         <(llvm-objdump -p "$OUT" | grep -oE "\bvk[A-Za-z0-9]+$" | sort -u)
```

**Normalise in the script, not in the file.** Re-saving the file as LF fixes today's clone and
nothing else — git will convert it again on the next checkout on the next machine. A
`.gitattributes` entry pinning those files to LF is a reasonable belt-and-braces addition, but the
script must not depend on it, because the script is what runs on an unknown machine.

## Generalised rule

> Any build check that compares text from the repo against text from a tool must normalise line
> endings first. Prefer `tr -d '\r'` at the comparison site over trusting repo configuration.

Sibling to the existing `strings -n 4` trap page (DOOM 2016, 2026-08-26): both are cases where a
**tooling default silently corrupted the input to a comparison** and produced a confident,
completely wrong conclusion. That pairing might be the actual page — "tool defaults that fabricate
false negatives" — with these two as its case studies.
