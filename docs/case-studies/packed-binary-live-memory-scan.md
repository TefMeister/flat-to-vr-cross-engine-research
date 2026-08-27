# RenderWare / Manhunt (2003) — when the file on disk lies, scan the live process instead

**Engine:** RenderWare 3.6 · **Render API:** Direct3D 8 · **Public VR conversion:** none exists.

Like the [id Tech 6 case study](./id-tech-6-dormant-stereo.md), this is a reconnaissance-shape entry,
not a study of someone else's finished adapter. It documents a DRM-remnant problem that looked, at
first, like a simple "find these known addresses" job — and the reason that approach failed cleanly,
in a way that generalizes to any packed or self-protecting binary.

The findings below are our own first-party static-and-live analysis of a legitimately-owned Steam
copy, published in full in the project repos linked at the bottom.

---

## The setup: a documented community fix, and a file that doesn't match it

Manhunt (2003)'s Steam release carries a well-known bug cluster — stuck gates, erratic AI, broken
saves — caused by leftover SecuROM anti-tamper checks that now misfire because Steam's own repackaging
stripped the *licensing* layer SecuROM originally protected, without removing the *code-level*
protection stub that layer depended on. A community tool, `Fire-Head/MHNoDRM`, documents 16 exact
virtual addresses where SecuROM originally IAT-hooked ordinary Windows APIs (`GetVersion`,
`GetLastError`, `IsBadReadPtr`, etc.) and substituted a fake return value the game's logic was written
to expect.

The natural first move is a static file patch: open `manhunt.exe`, go to each documented address,
confirm the expected `call [iat_slot]` opcode, patch it. **Every single one came back empty** — not
just the 16 documented sites, but a sanity check against guaranteed-to-exist calls
(`LoadLibraryA`, `ExitProcess`) found *zero* matches anywhere in the file for the correct opcode
pattern, ruling out an arithmetic mistake in the search itself.

## The diagnosis: the file on disk is not the code that runs

The real cause: `manhunt.exe`'s own `AddressOfEntryPoint` lands inside a section named `.bind` —
352KB, far larger than a normal bound-imports directory — sitting next to an obfuscated-named
section. Both are hallmarks of a still-active third-party protector stub, not compiled game code.
**The 16 call sites don't exist in readable form anywhere in the file on disk.** They only become
real, scannable `x86` instructions after the process unpacks itself into memory at startup — which a
static file read can never observe, no matter how careful the search.

This reframes the earlier "different build" hypothesis (a natural but wrong guess when documented
addresses don't match): a failed static check on a packed binary proves nothing about whether the
addresses are valid. It only proves the file is packed.

## The fix: scan the *process*, not the *file*

A DLL proxy already sitting in the target process (this project's standard `d3d8.dll` same-name
proxy) has a structural advantage no external tool has here: **`DLL_PROCESS_ATTACH` fires before the
exe's own entry point runs**, so code inside the proxy has full read/write access to the process's
own memory from the earliest possible moment, with none of the anti-attach restrictions that were
independently blocking an external debugger (`x32dbg` failed to attach, matching the same failure
signature seen on a different, Denuvo-protected project in this account — a live protector resisting
external debugger attach is standard behavior, not evidence of anything specific to either engine).

The working recipe: after a short delay post-`DLL_PROCESS_ATTACH` (to let the protector's own
unpacking stub finish), scan committed, readable+executable memory regions (confirmed via
`VirtualQuery`, not assumed) for the `FF 15 <iat-slot-VA>` / `FF 25 <iat-slot-VA>` opcode pattern for
each target API's real IAT slot. All candidates appeared within the first pass (seconds after
attach) — the unpacking here is fast, not the multi-minute worst case some later SecuROM revisions
are documented to use elsewhere. Cross-checked against the community tool's 16 documented addresses:
**16 of 16 matched exactly** once a fixed 2-byte labeling convention was accounted for (the community
tool cites the call instruction's operand; the scanner logs the opcode's own start).

## What generalizes

**1. A failed static-file check on a protected binary tells you the binary is protected — not that
your addresses, offsets, or build assumptions are wrong.** Before concluding "wrong version" or
"wrong build," check the entry point's containing section name and size; an oversized or
oddly-named section holding the entry point is a strong, cheap tell that you're reading packed
bytes, and no further static conclusion should be trusted until that is resolved.

**2. Any DLL already loaded into the target process can scan live memory where an external debugger
cannot reach.** A same-name proxy DLL (or any in-process hook point) initializes before the game's
own unpacking runs and is not subject to the process's own anti-attach defenses, because it *is*
part of the process rather than an outside observer of it. This turns "the debugger is blocked" from
a dead end into a reason to move the analysis in-process instead.

**3. Anti-tamper sabotage is often per-site, not a single blanket check.** Each of the 16 sites here
expected a *different* faked value or side effect (some a specific return value, one a write to a
global the caller polls) from a *different* real API. There is no one fix; each site needs its own
disassembly and its own targeted repair — usually cheapest as "force the branch the game takes when
the (now-impossible) expected value would have arrived," restoring the game's own intended path
rather than inventing new behavior.

**4. This is bug-fixing dead SecuROM cruft on an owned, DRM-stripped Steam copy, not defeating active
protection.** The community tool's own documentation states this explicitly, and it matches the
reasoning this library already applies to similar cases: study and reproduce the *technique* from
public write-ups, verify the finding independently against your own dump, write your own
implementation — never reuse the tool's code.

---

## Sources

**Our own first-party analysis** (static + live inspection of an owned copy; full evidence published):
- [`manhunt-2003-vr-engine-research`](https://github.com/TefMeister/manhunt-2003-vr-engine-research) — the consolidated engine dossier (§4, §11)
- [`manhunt-2003-vr-dev-archive`](https://github.com/TefMeister/manhunt-2003-vr-dev-archive) — raw recon evidence
- [`manhunt-2003-vr-external-research`](https://github.com/TefMeister/manhunt-2003-vr-external-research) — the SecuROM technique-family cross-check

**Community tool referenced (technique studied, no code reused):**
- [Fire-Head/MHNoDRM](https://github.com/Fire-Head/MHNoDRM) — documents the 16 addresses and the
  fake-return-value mechanism; explicitly states its fix "does not affect Steam copy protection in
  any way."

**Technical background:**
- [SecuROM 7 technical dissection](https://lostfilearchives.github.io/08/28/Dissection/) — a
  technique-description-only write-up (no crack/bypass content) of a later SecuROM revision, used
  here only to confirm the general packing architecture (stub wraps and decrypts the real program in
  memory), not as version-specific fact for this build.
- [PCGamingWiki — Manhunt](https://www.pcgamingwiki.com/) — documents the Steam-release bug cluster
  this DRM remnant causes.

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
