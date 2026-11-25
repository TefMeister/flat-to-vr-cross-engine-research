# /gr 2026-11-25: freezing threads to install hooks can deadlock on the process heap lock

Engine-agnostic: any hooking tool that suspends every other thread while it patches (kananlib's `suspend_threads`,
used by REFramework; UEVR uses kananlib too [inferred-static, from its dependency list, not re-checked today]).

kananlib commit 114de171 "Thread: Fix heap lock oversight" (cursey, 2026-11-17): a thread suspended inside
RtlAllocateHeap / RtlFreeHeap while it owns the heap's lock deadlocks the next allocation, and the suspender itself
allocates between SuspendThread calls (so do logging and hook setup). The fix takes the process heap lock last,
after the PEB and loader locks, with HeapLock(), so no thread can be frozen holding it. "Seen as an intermittent
REFramework startup hang in a game that was loading, i.e. allocating hard, while the world was frozen."
[reported 2026-11-25, the commit's own comment]

Symptom worth a library line: a hang with the game idle (no CPU) that comes and goes, most likely while a game is
loading and hooks are being installed. A build older than 2026-11-17 has it.

Where it touches us: visceral-re8-vr and visceral-re2-vr run REFramework `pd-upscaler` 76298bd (2026-03-11), without
the fix; RE8's topic `external-research/topics/2026-11-25-kananlib-heap-lock-fix-startup-hang.md`.

Source: https://github.com/cursey/kananlib/commit/114de171 ; REFramework "Deps: Update kananlib (fix startup
deadlock when freezing threads)", 2026-11-17, nightly 01441.
