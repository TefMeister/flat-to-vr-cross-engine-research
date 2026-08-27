# Never dispatch engine commands from a render-path hook

**From:** modding session, XIII (2003) VR, dev PC · **Date:** 2026-08-27
**Engine where it was proven:** Unreal Engine 2 / Direct3D 8 — but the
mechanism is engine-agnostic and worth a general page.

## The rule

If you hook a function to get a per-frame tick, **check whether that function
runs inside the renderer before you make it do anything but observe.** A hook
that is safe for *reading* state is not automatically safe for *acting* on it.

Reading per-frame data from a render-path hook: fine.
Calling into the engine's command/console/script system from one: crashes.

## What happened

Building an automation harness so an unattended session could drive the game,
the obvious per-frame tick was an existing camera hook — a function called once
per frame, on the game thread, with a live player object already in hand. It
looked ideal. Draining a command queue there faulted the game the first time a
command actually arrived:

```
General protection fault!
History: UGameEngine::Exec <- UGameEngine::Draw <- UWindowsViewport::Repaint
         <- UWindowsClient::Tick <- ClientTick <- UGameEngine::Tick <-
         UpdateWorld <- MainLoop
```

Read `<-` as "called from". The camera function is reached **from inside the
engine's Draw**. Executing a console command there re-enters the engine's
command system in the middle of rendering a frame.

Moving dispatch to the game-logic tick (the player controller's `Tick`, outside
the render path) fixed it outright, first try. The command system itself was
never the problem — *where it was called from* was.

## Why the mistake is easy to make

"Game thread + once per frame + has the player object" reads like a tick site,
and for **observation** it genuinely is. The trap is that a camera/view function
is naturally *called by the renderer* — computing the view is part of drawing —
so it satisfies every property you were checking for while sitting in the one
call stack where acting is unsafe. Nothing about the function's name or
signature says "you are inside Draw".

Engines that separate simulation from rendering (most of them) will have this
same hazard wherever a hook lands on the render side of that line — the specific
symptom will differ by engine, but "acting from inside the render path" is a
general class of fault, not a UE2 quirk.

## How to check before you act

1. **Get the crash stack, or read the call chain deliberately.** Unreal prints a
   guard stack naming every frame; other engines have equivalents. If the chain
   from `MainLoop` to your hook passes through anything named `Draw`, `Render`,
   `Present`, or `Repaint`, you are on the render path.
2. **Prefer a hook whose name is about simulation, not view** — an actor/entity
   `Tick`, the world update, the input phase.
3. If the only per-frame hook you have *is* on the render path, use it to
   **queue**, and drain from a simulation-phase hook. Never execute inline.

## The diagnostic habit that made this cheap to find

The first version logged each command **after** it completed. The crash
therefore left no record of which command or which call died, and the cause had
to be inferred from indirect evidence (telemetry stopping on an exact tick, the
process dropping to 0% CPU, nothing reaching the engine's log).

Changing to **log-before-the-call, flushed to disk**, so a fault names the
operation in flight, cost a few lines and turns "something crashed" into "this
exact call crashed". For any harness that drives a live game unattended, this is
worth doing from the first version — the fault you are instrumenting for is
precisely the one that stops you from collecting the evidence afterwards.

## Suggested placement

Fits either a general "harness & tick sites" page or the UE2 family page with a
generalisation note. The dispatch-site rule and the log-before-call habit are
independent of engine; the specific stack above is the UE2 illustration.
