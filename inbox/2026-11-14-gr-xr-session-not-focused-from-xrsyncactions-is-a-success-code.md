# /gr: XR_SESSION_NOT_FOCUSED from xrSyncActions is a success code; frameworks that log it as an error mislead (2026-11-14)

From `/gr` on visceral-re8-vr (topic:
https://github.com/TefMeister/visceral-re8-vr/blob/main/external-research/topics/2026-11-14-failed-to-sync-actions-session-not-focused-is-a-focus-change-not-a-fault.md).

- The OpenXR spec lists `XR_SESSION_NOT_FOCUSED` among `xrSyncActions`' success codes; while unfocused, every action
  reads inactive `[reported 2026-11-14, Khronos OpenXR 1.1 reference]`.
- REFramework's OpenXR layer logs any result other than `XR_SUCCESS` as "[VR] Failed to sync actions", and tracks
  focus only from polled events, so the line shows up at each focus change (system menu, overlay, headset off)
  `[inferred-static 2026-11-14]`.

Engine-agnostic lesson: when reading any VR layer's log, treat this code as "input paused, session unfocused", not a
fault; only a run of it every frame (the layer's state out of step with the runtime) is worth chasing `[hypothesis]`.
A layer of our own should check `result != XR_SUCCESS` with `XR_SUCCEEDED()` and handle the qualified codes apart.

Sources: https://registry.khronos.org/OpenXR/specs/1.1/man/html/xrSyncActions.html ; https://github.com/praydog/REFramework
