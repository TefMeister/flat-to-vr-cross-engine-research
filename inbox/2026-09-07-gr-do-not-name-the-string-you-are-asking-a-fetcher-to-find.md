# Never name the string you are asking an automated fetcher to find

**From:** `/gr` (estate sweep, 2026-09-07) · **For:** `/sr`, to curate into the library's research
method / techniques material

**Engine-agnostic.** This is about how we read the web, not about any engine.

## What happened

During RE Engine material research on 2026-09-07, a page fetch was asked, in effect: *"does this
forum thread mention `Detail_UVScale`, `DetailNormalMap`, `DetailMaskMap`, `DetailIntensity`,
`DetailNormalPower`?"*

The fetcher returned a **fabricated quote listing those five names as though they were the thread's
content**. A neutral re-fetch of the same URL showed the thread contains **none** of them — it is
five short posts about a broken download link `[measured 2026-09-07, n=1 page, 2 fetches]`.

The finding was caught and discarded before it reached any curated file, and only because a second,
differently-worded fetch of the same URL was made.

## Why it matters beyond that one page

This is a **false positive**, which is the dangerous direction. The research non-negotiables already
carry rule 7 — *"a negative from an automated fetch is not a negative"* — and the standard defence
against it is to put a known-present item in the query to prove the fetch could have found a
positive.

**That defence, applied naively, is exactly the thing that causes this failure.** Naming the target
string in the question gives the summarising model the answer to hand back, and a page that says
nothing can be reported as a page that confirms everything. So the two rules pull against each other
and the resolution needs stating:

- **To prove a fetch is capable of a positive**, use a control string you *already know is on the
  page* — not the string whose presence you are testing.
- **To test whether a string is present**, ask the fetcher an **open** question: *"list the parameter
  names this page mentions"*, *"what does this thread discuss?"* — and then look for your term in
  what comes back. Never *"does this page mention X?"*

## Suggested shape in the library

A short rule under the research-method material, roughly:

> **Do not name the string you are asking a fetcher to find.** An automated fetch asked "does this
> page contain X?" can answer "yes, here is X" about a page that does not contain X. Ask openly —
> "what does this page cover?" — and look for your term in the answer. Rule 7's capability check
> still applies, but the control string must be one you already know is present, never the one under
> test.

Pair it with the existing note on prompt injection, since both are cases of **fetched content, or
the fetching itself, producing text that is not what the page says**. The existing entry covers a
hostile page; this covers a compliant summariser. Same failure surface, opposite cause.

## Provenance and confidence

- `[measured 2026-09-07, n=1 page, 2 fetches]` on the specific incident: one fetch fabricated, a
  neutral re-fetch of the same URL did not.
- `[hypothesis]` on the generality — one observed instance. It is filed because the **cost asymmetry
  is severe** (a fabricated confirmation enters the record as fact and is acted on later) and the fix
  costs nothing, not because the frequency is established.
- This is the **second** confirmed case in this account of an automated read producing text a page
  did not contain; the first was the cloaked "AI instructions" served to fetchers on tcrf.net
  (2026-08-24), already recorded in the non-negotiables as rule 5.

No page content is reproduced here and nothing was cloned or downloaded. The affected research was
re-done with open-ended prompts and is written up in
[`visceral-re2-vr/external-research/topics/2026-09-07-the-shipped-detail-map-parameters-are-public-and-the-bc4-mask-is-a-channel-gamble.md`](https://github.com/TefMeister/visceral-re2-vr/blob/main/external-research/topics/2026-09-07-the-shipped-detail-map-parameters-are-public-and-the-bc4-mask-is-a-channel-gamble.md),
whose closing section records the incident in situ.
