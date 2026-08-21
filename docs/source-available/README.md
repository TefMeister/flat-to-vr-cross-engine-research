# When the source is open: VR source ports & SDK-based mods

The families elsewhere in this library (injectors, per-game native mods, generic drivers) all
fight a **closed** engine from the outside. There is a whole other world where that fight never
happens: games whose engine source is **publicly available** — via a GPL source release, an
open-source reimplementation, or a vendor SDK. There, the VR conversion is built **inside the
engine itself**, and the results are consistently the best in the entire flat→VR scene: true
per-eye stereo, full 6DoF, motion-controlled weapons, roomscale — no hooking, no injection, no
matrix archaeology.

If your target game is on this list (or on an engine with a source release), **this route beats
every other family in this library.** Check it first. All projects credited in
[`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).

---

## The three kinds of "open"

1. **GPL source releases** — id Software released the source of Doom, Quake 1/2/3, and Doom 3
   under the GPL; Wolfenstein and Jedi Knight engines followed the same path. Community source
   ports (GZDoom, dhewm3, ioquake3, Xash3D-FWGS…) keep them alive, and VR forks build on those.
2. **Open reimplementations** — an engine rebuilt from scratch for compatibility, e.g.
   **Xash3D-FWGS** (a GoldSrc-compatible engine) which made Half-Life 1 VR portable.
3. **Vendor SDKs** — Valve's **Source SDK 2013** ships enough engine/game source that a mod team
   can build a first-class VR version of Half-Life 2 as a "mod" in the official sense.

In every case the pattern is the same: **the camera, renderer, input, and game code are yours to
edit**, so VR becomes an engineering project, not a reverse-engineering one. The cost: these
engines are old (the games must be, for source to be out), and the GPL variants carry
**copyleft obligations** — VR forks must publish their source too (which is why this family is
so well documented).

## The landmark projects

### Team Beef — the standalone-headset port factory
**Team Beef** (DrBeef, Baggyg, and contributors) have turned GPL source ports into native
**Quest/Pico** VR conversions at scale: Doom 1/2 (**QuestZDoom**, from LZDoom/GZDoom, GPL-3.0),
Quake 1/2/3, Half-Life 1 (**Lambda1VR**, from Xash3D-FWGS, GPL-3.0), Doom 3 (**Doom3Quest**,
from dhewm3), Return to Castle Wolfenstein (**RTCWQuest**, LGPL-3.0), Jedi Knight: Outcast &
Academy (**JKXR**, GPL-2.0), plus Duke Nukem 3D, Blood, Shadow Warrior, Prey, and more — with
PCVR ports (RealRTCW-XR, WrathQuest) in progress. Their ports layer full motion-controller
gameplay, 6DoF weapon aim, and comfort options on top of the community engines they credit.
- [teambeefvr.com](https://www.teambeefvr.com/) ·
  [github.com/Team-Beef-Studios](https://github.com/Team-Beef-Studios) ·
  [SideQuest listing](https://sidequestvr.com/community/7/dr-beef) ·
  [patreon.com/teambeef](https://www.patreon.com/teambeef)

### Doom family on PC
- **GZ3Doom** (Christopher M. Bruns, continued by Fishbiter; GPL-3.0) — the original
  "classic Doom in VR" line: GZDoom modified for stereo 3D and Rift/OpenVR.
  [github.com/Fishbiter/gz3doom](https://github.com/Fishbiter/gz3doom) ·
  [hh79/gzdoomvr](https://github.com/hh79/gzdoomvr) (continuation fork)
- **QuestZDoom** brought the same idea standalone —
  [questzdoom.com](https://www.questzdoom.com/).

### Quake & Doom 3 on PC
- **Quake VR** (Vittorio Romeo; GPL-2.0) — Quake 1 with roomscale movement, two-handed
  weapons, hand interactions, melee — a showcase of how far "source in hand" lets you go.
  [github.com/vittorioromeo/quakevr](https://github.com/vittorioromeo/quakevr)
- **Quake2VR** — [github.com/q2vr/Quake2VR](https://github.com/q2vr/Quake2VR)
- **DOOM 3 BFG VR: Fully Possessed** (KozGit; GPL-3.0) — Doom 3 BFG with native
  Rift/OpenVR support; its VR gameplay layer was later ported into Team Beef's Doom3Quest.
  [github.com/KozGit/DOOM-3-BFG-VR](https://github.com/KozGit/DOOM-3-BFG-VR)
- Underlying non-VR ports these build on: **dhewm3** (GPL-3.0,
  [github.com/dhewm/dhewm3](https://github.com/dhewm/dhewm3)), **GZDoom**, **ioquake3**,
  **Xash3D-FWGS** ([github.com/FWGS/xash3d-fwgs](https://github.com/FWGS/xash3d-fwgs)).

### Source engine (Valve)
- **Half-Life 2: VR Mod** (Source VR Mod Team) — the flagship SDK-based conversion: free on
  Steam (requires owning HL2), built on **Source SDK 2013**; the team has said open-sourcing
  is possible in principle under the SDK's license terms once the code is releasable.
  [halflife2vr.com](https://halflife2vr.com/) · [FAQ](https://halflife2vr.com/faq/)
- **HL2VRU** ("Unleashed", Vittorio Romeo) — an unofficial fork adding VR-only interactions.
  [github.com/vittorioromeo/HL2VRU](https://github.com/vittorioromeo/HL2VRU)
- **L4D2VR** (sd805) — Left 4 Dead 2 VR with motion controls. L4D2's engine branch is *not*
  in the public SDK, so this one is **injection-based** rather than source-based — a reminder
  that "Source engine" ≠ "source available" for every title; the mod family straddles the line.
  [github.com/sd805/l4d2vr](https://github.com/sd805/l4d2vr)

## What a closed-engine project can still learn here

Even if your target's engine is closed (the usual case in this library), these projects are the
**best public reference for the gameplay half** of a VR conversion — the part injectors don't
solve either:

- **Weapon/hand decoupling** from the view camera (aim with controller, look with head).
- **Comfort options** that shipped and were validated by large player bases: snap/smooth turn,
  vignettes, seated height calibration, HUD projection distance.
- **Melee & physical interactions** (Quake VR's melee, Team Beef's gesture reloads).
- **UI re-projection** patterns for menus and HUDs designed for a flat screen.

They are also the cleanest demonstrations of the *end state* every engine adapter is trying to
approximate from outside: engine-true per-eye cameras at native frame rate.

## Sources

- Team Beef — [teambeefvr.com](https://www.teambeefvr.com/) ·
  [github.com/Team-Beef-Studios](https://github.com/Team-Beef-Studios) ·
  [SideQuest](https://sidequestvr.com/community/7/dr-beef) ·
  [Beginners Guide (Patreon)](https://www.patreon.com/posts/team-beef-guide-74479097)
- GZ3Doom — [github.com/Fishbiter/gz3doom](https://github.com/Fishbiter/gz3doom) ·
  [ModDB page](https://www.moddb.com/mods/gz3doom)
- QuestZDoom — [questzdoom.com](https://www.questzdoom.com/)
- Quake VR — [github.com/vittorioromeo/quakevr](https://github.com/vittorioromeo/quakevr)
- Quake2VR — [github.com/q2vr/Quake2VR](https://github.com/q2vr/Quake2VR)
- DOOM 3 BFG VR: Fully Possessed —
  [github.com/KozGit/DOOM-3-BFG-VR](https://github.com/KozGit/DOOM-3-BFG-VR)
- dhewm3 — [github.com/dhewm/dhewm3](https://github.com/dhewm/dhewm3)
- Lambda1VR (Xash3D-FWGS on Quest) —
  [github.com/Team-Beef-Studios/Lambda1VR](https://github.com/Team-Beef-Studios/Lambda1VR)
- Half-Life 2: VR Mod — [halflife2vr.com](https://halflife2vr.com/) ·
  [FAQ](https://halflife2vr.com/faq/)
- HL2VRU — [github.com/vittorioromeo/HL2VRU](https://github.com/vittorioromeo/HL2VRU)
- L4D2VR — [github.com/sd805/l4d2vr](https://github.com/sd805/l4d2vr)

Full credit list: [`../../ATTRIBUTION.md`](../../ATTRIBUTION.md).
