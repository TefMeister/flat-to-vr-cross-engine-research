# A legal-but-unnecessary API call is not evidence of a mechanism — the inverse of the silent no-op

Filed by: `/gr` (estate sweep, second pass 2026-09-04), for `/sr`.
Project instance: `doom-2016-vr/external-research/topics/2026-09-04-a-flush-count-on-coherent-memory-is-not-evidence-of-an-update-route.md`

## The general shape, which is what belongs here

The library already documents the **silent no-op**: a call that looks like it worked, produced no
error, and did nothing. This is its **inverse**, and it misleads in the opposite direction:

> **A call that is permitted but unnecessary will still appear in a trace, in volume, and it proves
> nothing about how the data actually got there.**

A counter is evidence that a call happens. It is not evidence that the call is the *mechanism*.
Instrumentation naturally produces the first and readers naturally hear the second, because a large
number feels like a finding — and the more calls counted, the more convincing the wrong conclusion
becomes.

## The instance that produced it, in one paragraph

A DOOM (2016, Vulkan / id Tech 6) session recorded 27,462 `vkFlushMappedMemoryRanges` on the memory
region where the camera copies live, and flagged it as contradicting an earlier dossier finding that
the camera buffer is `HOST_COHERENT` and therefore *not* updated through the flush path. There is no
contradiction. The Vulkan specification's Memory Allocation chapter says of
`VK_MEMORY_PROPERTY_HOST_COHERENT_BIT` that the host cache management commands
`vkFlushMappedMemoryRanges` and `vkInvalidateMappedMemoryRanges` **"are not needed to manage
availability and visibility on the host"** `[reported 2026-09-04, first-party source]` — and *not
needed* is not *not allowed*. No Valid Usage statement forbids the call. An engine that flushes
unconditionally, without branching on memory type, produces exactly that count while the flush does
no work: on coherent memory the write was already visible before the call was made.

So the flush count could not have discriminated between the two hypotheses it was being read as
evidence for. It was compatible with both.

## Why this is worth a library entry rather than a project note

The reasoning error does not depend on Vulkan. Any API with an **optional or advisory** call has the
same trap, and this family's projects sit on several:

- D3D9/11 `Flush`, `SetPredication`, redundant state-setting calls, and state blocks that re-apply
  values already set.
- `vkInvalidateMappedMemoryRanges` on coherent memory — the same case, other direction.
- Any "hint" API where the runtime is free to ignore the call entirely.

The discriminator is always the same and it is usually cheap: **find the property that decides
whether the call could have mattered, and read it.** In the Vulkan instance that is one field, the
memory type's `VkMemoryPropertyFlags` — coherent means the flush was ceremonial, non-coherent means
the count is real evidence. Reading that field costs nothing and settles which of two redesigns to
build.

## Suggested placement

Alongside **"Silent no-ops: verification that cannot see the failure"** in `docs/techniques/`, as
its stated inverse, so a reader meets both failure modes together: the call that did nothing while
looking successful, and the call that happened constantly while meaning nothing. A one-line
cross-reference in the Vulkan / id Tech engine page would carry the concrete instance.

## Source

- https://docs.vulkan.org/spec/latest/chapters/memory.html — Vulkan specification, Memory Allocation
  chapter (The Khronos Group). Quoted above; read online, nothing copied.
