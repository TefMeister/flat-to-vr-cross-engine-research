# Two engine-agnostic findings from six first live looks (2026-09-17, /lm, home PC)

Source projects: prey-2017-vr, deus-ex-mankind-divided-vr, borderlands-goty-vr, heavy-rain-vr,
bulletstorm-vr, burnout-paradise-vr. Detail: each repo's `modding-notes/2026-09-17-first-live-look.md`.

## 1. A dxgi.dll proxy can be called BEFORE its DllMain runs

- On Prey (CryEngine) and Deus Ex: Mankind Divided (Dawn), Windows' app-compat shim `AcGenral.dll`
  calls the proxy's `SetAppCompatStringPointer` export (dxgi ordinal 8) while the proxy is still
  being loaded, before `DllMain` has run `[verified-live 2026-09-17, n=2 games]` (x64dbg: return
  address in AcGenral, jump target 0).
- A proxy that fills its forwarding table in `DllMain` jumps to address 0 and the game crashes at
  start-up `[verified-live 2026-09-17, n=1]`.
- Loading the real `System32\dxgi.dll` at that moment fails with error 1168 `[verified-live 2026-09-17, n=2]`.
- Taking a lock around that load deadlocks: the real dxgi's load calls straight back into the
  proxy's export `[verified-live 2026-09-17, n=1]`.
- What worked: answer an export that arrives before the real dll is loaded with 0 (x64 only), do not
  mark it as seen, and retry the load on the next call and in `DllMain`, with no lock held. After that
  the real dll resolved 20/20 exports, and both games reached their menus `[verified-live 2026-09-17, n=2]`.
- Not seen on the same generator's d3d11/d3d9 proxies (Heavy Rain, Borderlands, Burnout) or on
  Bulletstorm's dxgi proxy `[verified-live 2026-09-17, n=4]`. It is probably shim-database-driven per
  exe `[hypothesis]`.
- Implementation: `staging/_shared/proxy-gen/gen_proxy.py` (TefMeister/staging).

## 2. Denuvo did not stop a passive d3d11.dll proxy (Burnout Paradise Remastered)

A full-export 32-bit `d3d11.dll` next to `BurnoutPR.exe` loaded under Denuvo plus the EA app. It
logged `D3D11CreateDeviceAndSwapChain`, and the game reached its main menu `[verified-live 2026-09-17, n=3]`.

## 3. "Keep these settings?" countdowns defeat unattended windowed-mode setup

- Heavy Rain and Prey both reverted in-game display changes because their confirm timers expired
  before synthetic input landed `[verified-live 2026-09-17, n=4]`.
- The config-file, registry and command-line routes all worked first time:
  - Prey: `system.cfg` `r_Fullscreen=0`
  - DXMD: `HKCU\...\Graphics` `Fullscreen=0`
  - Burnout: `config.ini` `WindowMode=1`
  - UE3 (Borderlands, Bulletstorm): `-windowed ResX= ResY=`
- UE3 note: Borderlands rewrote `ResX`/`ResY` in its user ini at launch, but honoured them on the
  command line `[verified-live 2026-09-17, n=1]`.
