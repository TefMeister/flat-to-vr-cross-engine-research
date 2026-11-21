# phunkaeg's xr-tape and xr-sim: an outside-the-mod OpenXR recorder, and a scripted headset-free runtime

From: an ordinary session on the Build PC, 2026-11-21 (Tefa found both links on Discord and asked "yes on all"
to: look, try xr-tape, write them into the library). Same author as the VR Modding Playbook already on
`docs/landscape/vr-modding-playbook.md`. Both starred, author followed (`claude-memory/github-thanks.md`).
For the page `docs/runtime-layers/README.md`, beside "Headset-free testing: OpenXR-Simulator and its forks".

## xr-tape: <https://github.com/phunkaeg/xr-tape> (NO LICENCE as of 2026-11-21: read and run only, no code taken)

- An **OpenXR API layer** that writes every frame's calls to a trace (NDJSON) and forwards everything
  unchanged; a Python checker then runs **20 checks** on the trace `[reported, from its README and docs/CHECKS.md]`.
- Why it generalises: it sits at the loader boundary, so it needs **no change to the mod**, covers every graphics
  API with one binary, and works on mods you did not write (UEVR included, as long as the process uses the Khronos
  loader). It checks the mod from outside, so a mod that is wrong about its own numbers cannot hide it.
- The checks worth naming in the library: eye order (both label and offset sign wrong, cancelling out); both eyes
  sent the same swapchain, slice and rectangle (mono in stereo costume); toe-in; eye spacing outside 45-80 mm;
  vertical or depth disparity over 2 mm; the two eyes submitted for different display times; a frame submitted
  with the view-state validity bits clear; more layers than the system allows (freezes the headset while the flat
  game runs on); declared FOV vs the FOV the runtime located; submitted pose vs located pose; and "no silent
  truncation" (a capped recording must not read as a complete one).
- **⭐ The rule worth folding in on its own:** for an **injector** (a mod drawing with the game's own frustum,
  UEVR-style), *declared FOV = located FOV is the defect*, not the pass: the game cannot have rendered with the
  runtime's FOV, so the declaration is a lie the runtime cannot detect; it shows up as wrong depth and scale. Their
  PreyVR measured Prey at 120 x 88.5 degrees, symmetric, against a Quest 3's l -54 / r +40 / u +44 / d -55. Second
  case: a mod that sometimes builds its own frustum owes an **in-process** check of declared vs rendered, because
  no layer can see the rendered projection; their PreyVR shipped exactly that slip and it read in the headset as
  three unrelated faults (wall-eyed, horizontal stretch, the sun swinging with head yaw) `[reported, docs/CHECKS.md]`.
- **⭐ Every check is proven able to fail:** a self-test builds a good trace, then injects one fault at a time and
  requires exactly the named checks to flip, no more. `[verified-numerically 2026-11-21, n=1]`: built x86 and x64
  on the Build PC (CMake + VS 2022, no warnings shown), self-test "SELFTEST PASS: 20 checks, 21 fault cases". Not yet
  run on a game; the first run is queued on the Silent Hill 2 board.
- Limits it states itself: OpenXR only (an OpenVR-only mod leaves an empty trace, which proves nothing); not
  programs that load a runtime DLL without the loader; no pixels in v1 (distinct eye resources, not distinct
  content); its own launcher refuses an elevated shell and a 32/64-bit mismatch, both of which make the loader
  skip a layer silently.
- Practical trap for us: a game started **through Steam** does not inherit our environment, so the layer's
  variables never reach it; start the exe directly (as our launchers' simulator mode already does).

## xr-sim: <https://github.com/phunkaeg/xr-sim> (MIT)

- A deterministic, headset-free OpenXR runtime, x86 and x64; D3D11, D3D12, OpenGL, Vulkan, headless, plus private
  opt-in D3D9 and D3D10 bindings (only for mods written to use them). Pictures only on D3D11; other APIs give
  telemetry only `[reported, its README and docs/RENDERERS.md]`.
- What it has that the OpenXR-Simulator forks do not, by its docs: frame-atomic **command scripts with asserts**
  (`.xrs` scenarios: head and hand poses, timed moves, buttons, `@assert sessionState eq FOCUSED`), frame stepping,
  **fault injection** (wait/begin/end failures, swapchain failures, instance loss), and a **measured Virtual
  Desktop focus-loss model**: while not FOCUSED, shouldRender=0, only layer-carrying frames count toward getting
  focus back, xrEndFrame throttled to about 87 ms; a mod whose loop submits zero-layer frames under shouldRender=0
  stays parked forever (their BioShock Infinite "raffle wedge"); re-submitting the last good layer set self-heals.
  That is a general lesson for any OpenXR mod on Virtual Desktop `[reported, scenarios/vdxr-park.xrs]`.
- Strict boot-time headset profiles: a malformed, non-finite, degenerate or oversized profile (over 16384) fails
  instance creation loudly instead of half-working. Matches two open faults on our side (`claude-memory/FAULTS.md`,
  the OpenXR Simulator's silent failure above 4096 wide and its ignored refresh rate).
- **Stereo acceptance in three legs** (their `docs/TESTING.md`): an app-owned per-eye production counter, the
  upstream wire trace (xr-tape), and image comparison; a post-compositor image pair is NOT a mono guard (their
  deliberate mono control stayed inside the normal image band while the per-eye rate fell from 90 to 0).
- A real fix worth knowing for anyone driving a runtime through a command file: a write time marked "seen" before
  the file was read dropped commands for good; read first, then mark (their CHANGELOG).

## Credit line for ATTRIBUTION.md

**phunkaeg**: *xr-tape* (no licence; read and run only) and *xr-sim* (MIT): the outside-the-mod OpenXR trace and its
checks, the injector FOV rule, the three-leg stereo acceptance, and the Virtual Desktop focus-loss model; no code
taken.
