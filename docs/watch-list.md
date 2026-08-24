# Watch list — where new flat→VR information appears

A living list of **public** sources this library's research sweeps check for new information.
Each entry says *why* it matters and *where* to look. Nothing here is downloaded or cloned to
check it — every source is read online (its own web page, releases page, or docs site) — see
[`CONTRIBUTING.md`](../CONTRIBUTING.md). All tools/people named are credited in
[`ATTRIBUTION.md`](../ATTRIBUTION.md).

This list is reviewed on a standing cadence (currently weekly); see the changelog at the bottom
for the last sweep date and what it found.

## Core injector / tool projects

| Source | Why it matters | Where |
|---|---|---|
| **UEVR** (praydog) | Covers Unreal Engine 4.8–5.x — the best-supported single case in the whole landscape. New features/engine support directly change what's turnkey vs. manual. | [github.com/praydog/UEVR/releases](https://github.com/praydog/UEVR/releases) · [docs.uevr.io](https://docs.uevr.io/) |
| **REFramework** (praydog) | Powers every RE Engine project we touch. New render-target / resource APIs, VR fixes, or documented internals could directly unblock open problems (e.g. the RT-GPU-backing question in our RE Village work). | [github.com/praydog/REFramework/releases](https://github.com/praydog/REFramework/releases) · [reframework.dev](https://reframework.dev/) · [issues](https://github.com/praydog/REFramework/issues) |
| **mutars** — starfield2vr / anvilengine2vr / Geo3D (Sergii Permiakov) | The reference non-Unreal engine adapters — proves the `IEngineAdapter` pattern generalizes. New titles/techniques here are the best signal for building our own from-scratch adapters. | [github.com/mutars](https://github.com/mutars) |
| **OldUnreal** — Unreal-testing / Unreal-PubSrc | Directly powers Unreal Gold VR (227k SDK, render-device contract). New SDK releases or engine fixes matter immediately. | [github.com/OldUnreal/Unreal-testing/releases](https://github.com/OldUnreal/Unreal-testing/releases) · [oldunreal.com forum](https://www.oldunreal.com/phpBB3/) |
| **vrframework** (Elliott Tate) | Source of the `IEngineAdapter` model and 10-milestone porting checklist we use conceptually. Updates may refine the checklist itself. | [github.com/elliotttate/vrframework](https://github.com/elliotttate/vrframework) |

## Community hubs (broadest signal, all engines)

| Source | Why it matters | Where |
|---|---|---|
| **Flat2VR community & Flat2VR Studios** | The hub of the whole hobby (150k+ Discord) — new mod releases, licensed-port announcements, and the pulse of what's being solved. | [flat2vrstudios.com/news](https://www.flat2vrstudios.com/) · [x.com/Flat2VR](https://x.com/Flat2VR) |
| **Road to VR**, **UploadVR** | Press coverage catches licensed-port news (Flat2VR Studios titles) and major tool releases before they reach niche forums. | [roadtovr.com](https://www.roadtovr.com/) · [uploadvr.com](https://www.uploadvr.com/) |
| **MTBS3D forums** | Long-running stereoscopic-3D/VR modding community — vorpX, geo-11, and generic-driver discussion lives here. | [mtbs3d.com/phpbb](https://www.mtbs3d.com/phpbb/) |

## Per-project relevance (checked with extra attention while these projects are active)

| Project | Engine | What to watch for | Where |
|---|---|---|---|
| Visceral RE2 VR / RE Village scope | RE Engine | `via.render.Mirror`, render-target/GPU-backing examples, new REFramework Lua script collections | REFramework issues/discussions, [alphazolam/EMV-Engine](https://github.com/alphazolam/EMV-Engine), REFramework Discord (web-indexed posts only) |
| The Evil Within | id Tech 5 | Any first public VR mod/injector attempt (none exists yet) | ModDB id Tech 5 page, Nexus Mods (Evil Within/Rage/Wolfenstein NOB-TOB), general search |
| Far Cry 2 | Dunia | Renderer architecture write-ups (e.g. the REAC 2023 Dunia shader-pipeline talk), any VR prior art | [enginearchitecture.realtimerendering.com](https://enginearchitecture.realtimerendering.com/) archive, Nexus Far Cry 2, general search |
| XIII (2003) | Unreal Engine 2 | UE2-era manual VR techniques (below UEVR's floor) | oldunreal.com, general UE2 modding forums |
| Unreal Gold | Unreal Engine 1 | 227k SDK updates, any new community render-device work | oldunreal.com, [OldUnreal GitHub org](https://github.com/OldUnreal) |
| Psychonauts | (see project engine-research) | — | — |

## Generic-driver ecosystem (fallback for older D3D9-and-older games)

| Source | Why | Where |
|---|---|---|
| **vorpX** | Commercial fallback for D3D9–12 games with no adapter. Feature/profile changes matter for older titles (Far Cry 2). | [vorpx.com/features](https://www.vorpx.com/features/) |
| **Vireio Perception** (cybereality) | Free/open alternative to vorpX — newly added to this library 2026-08-24. | [github.com/cybereality/Perception](https://github.com/cybereality/Perception) |
| **geo-11 / 3Dmigoto / Helix Mod** | DX11 stereo driver + shader-fix ecosystem; needs dgVoodoo2 to reach D3D9 games. | [helixmod.blogspot.com](https://helixmod.blogspot.com/) · [github.com/ThreeDeeJay/geo-11](https://github.com/ThreeDeeJay/geo-11) |
| **dgVoodoo2** (dege-diosg) | The D3D8/9→11 wrapper that unlocks geo-11 for older games. | [github.com/dege-diosg/dgVoodoo2/releases](https://github.com/dege-diosg/dgVoodoo2/releases) |

## Engine architecture talks (primary-source technical background)

| Source | Why | Where |
|---|---|---|
| **Real-Time Engine Architecture Conference (REAC)** archive | Developer-authored architecture talks (e.g. Ubisoft's own Dunia shader-pipeline talk) — legitimate primary sources for renderer internals without reverse-engineering risk. | [enginearchitecture.realtimerendering.com](https://enginearchitecture.realtimerendering.com/) |
| **GDC Vault** (free talks) | Similarly authoritative for engines whose developers have publicly spoken about rendering/VR. | [gdcvault.com](https://www.gdcvault.com/) |

## How a sweep works

1. Check each source above for anything new since the last sweep.
2. For anything relevant: verify it's genuinely public, summarize the *technique* in our own
   words, and note exactly where it came from.
3. Add it to the right doc here with a link, and add/update the source in
   [`ATTRIBUTION.md`](../ATTRIBUTION.md).
4. Nothing is downloaded or cloned to do this — see [`CONTRIBUTING.md`](../CONTRIBUTING.md) rule 1.
5. If nothing new turns up, that's a valid, useful result — it confirms the landscape is stable.

## Sweep log

- **2026-08-24:** Added Vireio Perception, Flat2VR Studios' August 2026 VR Games Showcase
  titles, and the Dunia REAC 2023 shader-pipeline talk. Confirmed no change needed for id Tech 5,
  Dunia VR prior art, or XIII/Unreal Gold engine identification (already accurate).
