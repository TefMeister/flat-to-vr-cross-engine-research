# Two hard lessons from a first live run: WC memory reads, and validating your instrument

Supersedes: the "measure input backends against a no-input control" technique section contributed
2026-08-31 (docs/techniques/README.md) — **the control idea stands, but it needs a validity check
in front of it.** See part 2.

**From:** modding session (`doom-2016-vr`), 2026-08-31
**Confidence:** `[measured 2026-08-31]` — both observed in a live 43,028-frame gameplay session.

---

## 1. Never CPU-scan mapped GPU memory in place — it is write-combined

**The measurement:** scanning ~96 MB of `HOST_VISIBLE` Vulkan memory for candidate matrices took
**3 minutes 45 seconds** and froze the game solid for the duration (no frames presented). Effective
read throughput was about **430 KB/s** — roughly three orders of magnitude below normal RAM.

**The cause:** `HOST_VISIBLE` / upload memory is typically **write-combined**. WC is designed for
streaming CPU *writes* to the GPU; CPU *reads* out of it bypass cache and defeat prefetching
entirely. The scan compounded it by doing ~24 million *small strided* reads (4-byte stride, 64-byte
spans), which is close to the worst possible access pattern for WC.

**The fix, which is cheap and general:**

1. **One bulk sequential `memcpy` of each region into ordinary cached RAM, then scan the copy.**
   Sequential bulk reads are the one thing WC memory does acceptably.
2. **Widen the stride to the alignment you actually need.** Uniform-buffer matrices are at least
   16-byte aligned, so a 4-byte stride does 4× redundant work for nothing.
3. **Cheap reject first.** A basis vector is unit length, so six multiplies eliminate almost every
   offset before any expensive check runs. NaN fails the comparison for free.
4. **Order regions by flush count.** A per-frame uniform buffer flushes thousands of times; a
   static upload flushes once. When a budget runs out, spend it where the camera actually is. (In
   DOOM: one region had 27,907 flushes against another's 2,983 and zero for the rest.)

**Applies to:** any D3D/Vulkan project that hunts for matrices in mapped memory — which is most of
the projects in this estate. Worth a technique page of its own; the "how do I find the view matrix"
approach is common here and this makes the difference between a two-second scan and a four-minute
freeze.

---

## 2. Validate the instrument before you trust the experiment

My own 2026-08-31 drop argued: measure input backends against a **no-input control run**, never
against the API's return value. **That reasoning is still right and I am not withdrawing it.** But
it is incomplete in a way that cost a live session, so the library page needs a step in front of it.

**What happened:** the control-based probe reported **"no clear reaction"** for the backend that had
just **walked the player fifteen metres across the map**. Control 274 changed matrices, that backend
235, another 134 — both *below* the control.

**The tell, and the generalisable rule:** a backend that does nothing should score **the same** as
the control, not less. **Scoring below your control means your instrument is measuring noise, not
your variable.** That check costs nothing and would have caught it immediately.

**Why the instrument was invalid:** the metric counted "did the bytes at these addresses change",
but the addresses lived in **per-frame dynamic/ring buffers**, whose contents are reused for
unrelated data every frame. It was measuring buffer recycling.

**What settled it in two seconds:** a screenshot. The game's own on-screen waypoint distance
(271.9 m → 257.0 m) is unambiguous ground truth. This is the failure `capture-window.ps1`'s header
already warns about — *"the cheapest way to get it wrong is to infer state from a derived number
instead of looking"* — and I walked into it while holding the tool built to prevent it.

**Suggested wording for the page:**

> Before trusting any derived metric, establish that it responds to a change you can independently
> confirm. Run one positive control — an input you can verify by eye or by an on-screen readout —
> and check the metric moves. If it does not, fix the instrument before running the experiment. And
> if a null condition ever scores *worse* than your control, the instrument is measuring noise.

A near-miss worth recording too: the replacement metric (mean pixel difference between before/after
screenshots) was *also* misleading, reporting 0.93 % for a working backend — because by then the
player was jammed against a wall, so the input genuinely changed little. Looking at the two images
resolved it instantly. **Two different derived metrics failed in one session; looking never did.**
