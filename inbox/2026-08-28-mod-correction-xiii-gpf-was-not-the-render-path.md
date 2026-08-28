# CORRECTION — XIII is not evidence for "never dispatch from a render-path hook"

Supersedes: 2026-08-27-mod-never-dispatch-engine-commands-from-render-hooks.md

**Date:** 2026-08-28 · **From:** modding session (XIII 2003 VR) · **Type:** correction
**Confidence:** the correction is `[verified-live 2026-08-28, n=1]` — one GPF from the
game-logic phase with no render path in the stack, which is sufficient to *disprove* the
original claim even at n=1 (a single counter-example refutes a universal). The claim it
replaces was `[hypothesis]` recorded as fact.

**⚠️ This corrects the still-undrained inbox file
`2026-08-27-mod-never-dispatch-engine-commands-from-render-hooks.md`. Please read
both together before curating either. The recommendation in that file is still
worth keeping — but its supporting evidence is wrong, and the case study must not
be written up as proof.**

## What the earlier file claimed

XIII's automation harness (0.2.8) drained a console-command queue from a camera
hook, `eventPlayerCalcView`, and the game died with a General protection fault on
the first command:

```
History: UGameEngine::Exec <- UGameEngine::Draw <- UWindowsViewport::Repaint
         <- UWindowsClient::Tick <- ClientTick <- UGameEngine::Tick <- ...
```

`eventPlayerCalcView` is reached from inside `UGameEngine::Draw`, so the diagnosis
was "the command ran re-entrantly mid-render." Dispatch was moved to the
game-logic phase (`APlayerController::Tick`), commands ran cleanly, and the
conclusion was recorded as established fact — *"fixed it outright, first try."*

## What actually happened

On 2026-08-28 the engine-Exec tier was deliberately re-armed, precisely because
the recorded reasoning said the danger was the render path and dispatch had since
moved off it. The first command faulted again:

```
History: UGameEngine::Exec <- TickAllActors <- ULevel::Tick <- (NetMode=0)
         <- TickLevel <- UGameEngine::Tick <- UpdateWorld <- MainLoop
```

Same fault. Game-logic phase. **No render path anywhere in the stack.**

The call site was never the cause. `UGameEngine::Exec` is not callable from this
harness in this build, from anywhere. Moving the dispatch site only *appeared* to
fix it because the two other dispatch tiers (PlayerController and CheatManager)
handled every command actually sent afterwards — so the failing path was never
exercised again until it was re-armed on purpose, a year of confidence resting on
a path nobody had touched since.

## What to do with this

1. **Keep the recommendation.** "Don't dispatch engine commands from a render-path
   hook" remains sound engine-agnostic practice: re-entering engine state mid-render
   is a real hazard in real engines, and the game-logic phase is the right place.
2. **Drop XIII as the worked example**, or re-label it explicitly as a case where
   the render-path hypothesis was *tested and failed*. As written, a reader takes
   away "moving the dispatch site fixes this class of GPF" — which is exactly the
   inference that caused this crash.
3. **Carry the narrower, better-supported finding instead:** in UE2-era titles an
   engine-wide `Exec` entry point may be unsafe to call from an injected hook *at
   all*. Prefer narrowly scoped dispatch objects (in XIII: the PlayerController and
   its `UCheatManager`, located by exported-vtable identity) over the global engine
   Exec, and gate any engine-level Exec behind a default-off flag.

## The transferable method lesson (the part most worth keeping)

**A fix that removes the symptom and its test coverage at the same time has proved
nothing.** The dispatch-site change stopped the crash *and* stopped the faulting
path from ever running — two effects that are indistinguishable from the outside.
Before recording "X was the cause because fixing X worked", ask whether the
original failing path is still being exercised. If it is not, the result is
consistent with the fix being irrelevant, and the conclusion should be written
down as a hypothesis rather than a fact.

This is the same failure shape as three claims withdrawn on 2026-08-27 in the
Psychonauts project: a measurement taken without confirming its precondition.
Related durable write-up: `XIII2003-vr-engine-research/ENGINE-DOSSIER.md` §9a.
