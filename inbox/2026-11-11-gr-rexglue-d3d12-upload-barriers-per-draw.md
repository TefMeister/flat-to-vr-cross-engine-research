# /gr 2026-11-11: in ReXGlue's D3D12 backend, every draw that finds changed guest memory costs a whole-buffer barrier pair

**From:** `/gr the-darkness-vr`, 2026-11-11. **For:** `docs/engines/rexglue-static-recomp.md` (shared by the-darkness-vr
and condemned-2-vr). Engine-agnostic within ReXGlue: it is shared SDK code, not game code.

**The finding:**

- `[inferred-static 2026-11-11, read online from rexglue-sdk v0.10.0 and development]` For each draw the D3D12 command
  processor requests the vertex-fetch (and memexport) ranges from the shared-memory buffer; if a page changed,
  `D3D12SharedMemory::UploadRanges` transitions the WHOLE buffer to `COPY_DEST`, submits the barriers, copies, and the
  draw transitions it back for reading. A barrier pair on the one buffer every draw reads, mid-stream. Inherited from
  Xenia.
- `[measured 2026-11-10]` in the-darkness-vr (Nsight GPU trace, one eye, simulator): ~560 small memory uploads an eye,
  ~10 % of card time, and a size-independent ~2-3 microseconds of card time a draw. That the barriers are part of that
  per-draw cost is `[hypothesis]`.
- `[reported 2026-11-11]` Two open rexglue-sdk pull requests attack it, **Vulkan only**: #536 (datamothra) records
  uploads in a slot ahead of the open render pass when no draw in it reads those pages (Tekken 6 / M1 Max: ~125 to 56
  passes a frame, ~21 to 60 frames shown); #549 (mkalkhoven-po, for #548) stops write-protecting pages rewritten every
  frame and compares them with a shadow copy instead (NFS Carbon / RTX 5080: race 87 to 147 FPS), with the D3D12 half
  named and left out.
- `[inferred-static 2026-11-11]` Separately: the code generator's `skip_msr` manifest key (present since v0.10.0)
  removes the global "interrupts off" lock around every `lwarx`/`stwcx.` step, which the-darkness-vr found costing a
  third of the graphics thread's time (b024, note 2026-11-10g) and removed by a post-generation script instead.
  Upstream keeps the lock (now `rex::platform::InterruptMask`, a recursive mutex).

**Suggested change:** a short "per-draw costs in the D3D12 backend" paragraph on the rexglue page: the upload/barrier
pattern, the two Vulkan PRs as the known designs, and `skip_msr` vs keeping the lock, crediting the PR authors and
naming the-darkness-vr (`TefMeister/the-darkness-vr`) as where the numbers came from.

Full topic: `the-darkness-vr/external-research/topics/2026-11-11-rexglue-sdk-per-draw-uploads-and-0-11-nightlies.md`.
Links: <https://github.com/rexglue/rexglue-sdk/pull/536>, <https://github.com/rexglue/rexglue-sdk/pull/549>,
<https://github.com/rexglue/rexglue-sdk/blob/v0.10.0/src/graphics/d3d12/shared_memory.cpp>
