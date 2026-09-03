# A packed `.text` makes every static scan a false negative — and the check that catches it costs seconds

**From:** `/gr` (2026-09-03, estate sweep)
**Suggested home:** a method page — this belongs beside the "positive control" discipline, not on an
engine family page. It is renderer- and engine-agnostic; it is about *reading a Windows game binary
at all*.

## The pattern, seen twice in the estate now

Two projects have now had a static scan of a shipped executable return a clean zero **for a thing
that is definitely present**, because the code section was encrypted at rest:

- **Manhunt (2003)** — packed build; the route that worked was dumping `.text` from the running
  process and scanning the dump.
- **Alice: Madness Returns (2011)** — an NVAPI id scan returned zero occurrences for all three
  target ids *including the `NvAPI_Initialize` positive control*, on 2026-09-02.

In both cases the scan was correct, the tooling was correct, and the answer was worthless. **A
negative from an unreadable section looks exactly like a real negative.**

## The three-line identification test

Before trusting any static scan of a game exe, check the code section is actually code. All three
are cheap and none require running anything:

1. **Section names.** A section named **`.bind`** is Steam's own DRM stub (SteamStub, applied at
   upload time). `.vmp0`/`.vmp1` is VMProtect; `.themida`/`.winlice` is Themida. A wrapper section
   is usually the last one in the file.
2. **Entry point location.** If the PE entry point does **not** land inside `.text`, something else
   runs first and `.text` is very likely encrypted until it does.
3. **Two cheap statistics on `.text`:** entropy at or near **8.00**, and **zero `CC` padding runs**.
   Real MSVC code sections always carry `CC` (int3) alignment padding between functions; a section
   with none is not code as stored.

Alice hit all four tells at once (`.bind` present, entry point at `01661310` inside it, entropy
8.00, no `CC` runs). `[measured 2026-09-02, on that project]`

## What to do about it, cheapest first

- **If the wrapper is SteamStub**, unpacking is a **static** step, not a runtime one. Two
  open-source unpackers exist — [Steamless](https://github.com/atom0s/Steamless) (variants 1–3,
  32- and 64-bit) and [Steamstub-v3-Unpacker](https://github.com/GHFear/Steamstub-v3-Unpacker) —
  both explicitly for software you own. Run over a **copy**; never overwrite the shipped exe. This
  keeps the task in the "no game running" tier.
- **Otherwise, dump from memory.** Every wrapper of this class decrypts into memory before handing
  over control, so a dump of the running process yields plaintext code. This is the Manhunt route,
  and it is the universal fallback when no unpacker recognises the build.
- **Expect the unpacked file not to launch.** It generally will not run standalone, because the game
  still expects the environment the stub set up. That is an expected outcome, not a failed unpack —
  we want a readable section, not a second way to start the game. `[reported 2026-09-03]`

## Why this is worth a page rather than a footnote

The estate already has the right instinct — **every scan carries a positive control** — and on Alice
that discipline is the only reason the result was read as "blocked" instead of "the game uses
Automatic stereo". This note is the other half of it: *when the control fails, the next question is
not "is my scanner broken" but "is this section encrypted", and that question has a three-line
answer.* Recording the tells once, centrally, saves every project rediscovering them.

There is also a scheduling consequence worth stating plainly, since the estate now tags work by what
it requires: **identifying the wrapper can move a task from `[FLAT]` back to `[PD]`.** On Alice it
did exactly that — the blocked scan was re-tagged from "needs the game running" to "static, do it
now" purely by naming the packer.

## Sources

- [atom0s/Steamless](https://github.com/atom0s/Steamless) — supported SteamStub variants, purpose,
  licence, own-your-games stipulation.
- [GHFear/Steamstub-v3-Unpacker](https://github.com/GHFear/Steamstub-v3-Unpacker) — v3 unpack and
  rebuild behaviour, keep-`.bind` option.
- [Adam Hlt, "Cube World Reversing — Unpack the game"](https://adamhlt.com/cube-world-reversing-unpack-the-game/)
  — the `.bind` section, entry-point redirection into it, `.text` encrypted at rest and decrypted at
  runtime, and the two practical routes.
- The estate's own evidence: `alice-madness-returns-vr/engine-research/ENGINE-DOSSIER.md` §6 and §11
  (the measurement), and its
  [2026-09-03 topic](https://github.com/TefMeister/alice-madness-returns-vr/blob/main/external-research/topics/2026-09-03-the-bind-section-is-steamstub-and-a-public-unpacker-restores-text.md)
  (the identification).
