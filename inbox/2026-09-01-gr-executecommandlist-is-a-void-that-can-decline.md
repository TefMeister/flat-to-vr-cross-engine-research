# `ExecuteCommandList` returns `void` and can silently decline to run — a documented silent no-op for the collection

**From:** `/gr`, 2026-09-01 (estate-wide sweep)
**Adds to** `docs/techniques/README.md` § *"Silent no-ops: verification that cannot see the failure"*.
No existing claim is contradicted — this is a new instance, and it is a better one than most because
it is **documented by the vendor** rather than discovered by us or found in someone's bug tracker.

## The finding

Microsoft's reference for `ID3D11DeviceContext::ExecuteCommandList` states, verbatim:

> *"This method performs some runtime validation related to queries. Queries that are begun in a
> device context cannot be manipulated indirectly by executing a command list (that is, Begin or End
> was invoked against the same query by the deferred context which generated the command list). If
> such a condition occurs, **the ExecuteCommandList method does not execute the command list.**
> However, the state of the device context is still maintained, as would be expected
> (`ClearState` is performed, unless the application indicates to preserve the device context
> state)."*

**The method returns `void`.** There is no `HRESULT`, no out-parameter, nothing to check. So a whole
command list can be submitted, silently discarded, **and still clear the context state on its way
past** — leaving a frame that renders wrongly with no error raised anywhere.

For a VR hook that injects, reorders or duplicates command-list execution around a game that uses
D3D11 queries — occlusion, timestamps, predication, all ordinary in a renderer of this class — the
symptom is a missing or wrongly-stated pass. The natural diagnosis is *"my patch is wrong"*, and the
real answer is *"my command list was never run."* That is squarely the shape the silent-no-ops page
exists to catalogue.

**The guard is cheap and specific: run bring-up with the D3D11 debug layer enabled.** The runtime
validation quoted above is exactly what the debug layer surfaces, so it converts an invisible discard
into a message. Worth doing *before* chasing a patching bug, not after.

## A second, quieter hazard on the same page, worth a line beside it

`ExecuteCommandList`'s `RestoreContextState` parameter: passing **`FALSE`** *"causes the target
context to return to its **default state** after the command list executes"* — and the documentation
recommends `FALSE` for performance (*"Applications should typically use FALSE"*). So on most real
engines, anything running after an execution inherits **nothing**: no render targets, no constant
buffers, no shaders. A per-eye pass that assumes it inherits the state the previous list left will
fail in a way that reads as a patching bug rather than a state-management one.

## Why it is worth adding as a distinct entry

The existing entries on that page are about verification that *reports success wrongly* (an empty
optional satisfying a `!= 0` test; a `false` return stopping being honoured; a read-back compared
against zero). **This one has no verification surface at all** — the API gives you nothing to check,
by design. That is a slightly different and worse category, and it argues for a general rule the
page could state outright:

> **When an API returns `void`, ask what it does on failure before you rely on it.** A function that
> cannot report failure has not thereby become infallible; it has moved the failure somewhere you
> have to go looking for it. Where a debug layer or validation layer exists, that is where it went.

That generalises past D3D11 — Vulkan validation layers, and any `void` submit/execute call, sit in
the same shape.

## Where it came from

Turned up while researching `the-evil-within-vr`'s live open risk (its renderer records the world on
worker-thread deferred contexts, so command-list semantics are on its critical path). Full write-up,
including why per-eye *re-execution* of a command list cannot work on an engine with per-draw pooled
constant buffers:
[`the-evil-within-vr/external-research/topics/2026-09-01-command-list-reexecution-cannot-do-per-eye-and-two-documented-hazards.md`](https://github.com/TefMeister/the-evil-within-vr/blob/main/external-research/topics/2026-09-01-command-list-reexecution-cannot-do-per-eye-and-two-documented-hazards.md)

## Sources

- [`ID3D11DeviceContext::ExecuteCommandList` — Microsoft Learn](https://learn.microsoft.com/en-us/windows/desktop/api/D3D11/nf-d3d11-id3d11devicecontext-executecommandlist)
- [Command List — Direct3D 11, Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3d11/overviews-direct3d-11-render-multi-thread-command-list)
