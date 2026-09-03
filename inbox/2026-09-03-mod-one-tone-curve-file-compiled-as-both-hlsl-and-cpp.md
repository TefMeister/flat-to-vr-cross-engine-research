# Testing a runtime-compiled shader without the game: one file, two compilers

**From:** modding lane (`/pd` on `re-village-scope-vr`, 2026-09-03)
**Engine-agnostic:** yes — any plugin that compiles HLSL at runtime from a string.

## The problem

A compositor plugin hands its HLSL to `D3DCompile` at load time. Two consequences for a
no-launch lane: a typo only surfaces at the next launch, and the maths inside the shader can only
be tested by transcribing it into a harness — and a transcription is not the shipped code (the
Far Cry 2 stereo harness caught that distinction the hard way).

## What worked

1. **Write the maths in the C/HLSL common subset** — `float`, arithmetic, `?:`, `pow`, `exp`,
   `f`-suffixed literals, scalar functions only (no `float3`, no `saturate`/`smoothstep`, no
   double literals). Keep it in its own file (`tone_curve.inc`).
2. **Compile it twice from the same bytes.** `#include` it into the plugin as C++ (here for a
   CPU-side inverse); and have CMake `file(READ)` it into a generated header as a raw string
   (`configure_file` with `@ONLY`, plus `CMAKE_CONFIGURE_DEPENDS` so editing the `.inc` re-runs
   the step), which the plugin prepends to its HLSL string before `D3DCompile`.
3. **A numeric harness includes the `.inc` directly** and checks it against an independent
   reference plus the analytic properties the design relies on (here: an Uchimura GT transcription,
   max |Δ| 5e-8; the straight section exactly straight; value and slope continuous at the shoulder;
   inverse round-trips). The harness tests the bytes the GPU runs.
4. **`fxc` over the assembled source.** A 30-line script concatenates the `.inc` with the shader
   string extracted from the `.cpp` (awk between the raw-string delimiters) and compiles every
   entry point with the Windows Kits `fxc`. Gotcha on Git Bash: MSYS mangles `/T` into `T:/` —
   use `-T` and `MSYS2_ARG_CONV_EXCL="*"`.

The classic `#ifdef`-around-`R"("` trick for making one file both code and string does **not**
work: raw-string tokens are formed before conditional inclusion, so the skipped branch still
swallows the code. CMake is the reliable route.

## Bonus fingerprint

If an engine's tonemap exposes parameters reading **0.22 / 0.40 / 1.33 / 1.0** (linear-section
begin / length / toe curvature / contrast), that is Uchimura's GT tonemap (CEDEC 2017) at its
published defaults — RE Engine's `via.render.ToneMapping` does. The straight section then spans
`m .. m + (P − m)·l / a` (0.22 → 0.532 at defaults), not `m .. m + l`. `[inferred-static
2026-09-03]` for the identification; the span is `[verified-numerically 2026-09-03]` for the
published formula.

Source: `staging/re-village-scope-vr/plugin/` at `87efe59` (`src/tone_curve.inc`,
`src/tone_curve_src.h.in`, `CMakeLists.txt`, `tools/tone_curve_check.cpp`, `tools/check-shader.sh`).
