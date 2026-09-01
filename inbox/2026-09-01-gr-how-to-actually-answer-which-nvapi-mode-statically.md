# How to actually answer "which 3D Vision mode does this game request?" — statically, offline

Filed by: `/gr`, 2026-09-01, from `alan-wake-vr` (verified on that game the same day)
Suggested home: `docs/techniques/README.md`, the **Automatic vs Direct** section — as the missing
procedure, not a new claim

## The gap this fills

That section already says the right thing: a binary referencing NVAPI stereo is not a binary that
renders two eyes, and **which driver mode a title requests separates the two statically, before
anything is launched**. What it does not say is *how you actually perform that check* — and it is
less obvious than it sounds, because the first thing anyone tries fails.

## Why the obvious method fails, and what works

**NVAPI has no import table entries to read.** A game links `nvapi.lib`, but every function is
resolved at runtime through a single exported dispatcher, `nvapi_QueryInterface`, taking a **numeric
function ID**. So `NvAPI_Stereo_SetDriverMode` never appears in the game's imports and never appears
as a string — the name exists only in the SDK header, at compile time.

The three facts that make the check work anyway:

1. **Each wrapper is findable as a `push imm32` of its ID**, immediately before the
   `nvapi_QueryInterface` call. Counting those sites in the renderer binary counts the NVAPI
   functions the game actually calls — no debugger, no launch.
2. **The shipped driver cannot give you the id→name mapping.** The IDs occur in `nvapi.dll` /
   `nvapi64.dll`, but the name table is stripped, so nothing on the machine tells you which is
   which. This is the step that traps people.
3. **NVIDIA publishes the whole table in the open** — `nvapi_interface.h` in the public NVAPI
   repository is literally the id→name list `nvapi_QueryInterface` dispatches through. So the
   mapping is a web read, and the check is completed entirely offline from the game.

The IDs worth having to hand: `NvAPI_Initialize` `0x0150E828`,
`Stereo_CreateHandleFromIUnknown` `0xAC7E37F4`, `Stereo_Activate` `0xF6A1AD68`,
`Stereo_SetSeparation` `0x5C069FA3`, `Stereo_Enable` `0x239C4545`, **`Stereo_SetDriverMode`
`0x5E8F0BEC`**. `[reported 2026-09-01]` — from NVIDIA's published table, corroborated against an
independent list.

## How to read the result

- **`SetDriverMode` called ⇒ Direct.** The application splits the draws; a native two-eye path
  plausibly exists and is worth chasing.
- **`SetDriverMode` absent ⇒ Automatic** (it is the default and must be changed *before* device
  creation, so a game that never calls it can never be in Direct). Its stereo symbols are the
  correction layer over the driver's work, exactly as the section already warns.
- **`Stereo_Enable` absent means nothing.** It is a persistent, driver-wide user setting, not a
  per-session call — well-behaved games leave it alone. Do not read its absence as evidence.

## Worked example

Alan Wake's `renderer_sf_Win32.dll`: `Initialize` 4 callers, `CreateHandleFromIUnknown` 2,
`Activate` 1, `SetSeparation` 1, **`SetDriverMode` 0**, `Enable` 0 `[inferred-static 2026-09-01]`.
A tidy Automatic-mode init sequence and no request for Direct — which retired that project's
native-stereo shortcut on static evidence alone, before anything was launched.

## Why it generalises

Nothing here is about Remedy's engine. It applies to **any Windows title of the 3D Vision era**
(roughly 2008–2013) that touches NVAPI at all, several of which are on this account across
completely different engines. The check costs one binary scan and one web read, and it can retire
or promote a native-stereo lead before a single launch.

## Sources

- NVIDIA, `nvapi_interface.h` (the published id→name dispatch table) —
  https://github.com/NVIDIA/nvapi/blob/main/nvapi_interface.h
- NVIDIA, `nvapi_lite_stereo.h` (`NvAPI_Stereo_SetDriverMode`; Direct vs Automatic) —
  https://github.com/NVIDIA/nvapi/blob/main/nvapi_lite_stereo.h
- jNizM, `NVIDIA_NvAPI` — `info/NvAPI_IDs.txt`, an independent ID list used as a second source —
  https://github.com/jNizM/NVIDIA_NvAPI/blob/master/info/NvAPI_IDs.txt
- Full write-up: `alan-wake-vr/external-research/topics/2026-09-01-nvapi-function-ids-confirmed-against-nvidias-own-table.md`
