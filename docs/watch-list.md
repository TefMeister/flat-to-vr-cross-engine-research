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
| **PCVR Central** (Camracks, added 2026-08-24) | A non-rehosting directory of ~900 PC VR mods across ~968 games with quality/freshness labels and a Steam-library checker — a fast cross-check for "does a mod already exist" and for landscape/framework coverage before starting new adapter work. | [pcvrcentral.com/mods](https://pcvrcentral.com/mods) · [pcvrcentral.com/library](https://pcvrcentral.com/library) |

## Per-project relevance (checked with extra attention while these projects are active)

| Project | Engine | What to watch for | Where |
|---|---|---|---|
| Visceral RE2 VR / RE Village scope | RE Engine | `via.render.Mirror` and `via.render.layer.Scene`, render-target/GPU-backing examples, new REFramework Lua script collections — and **REFramework Lua-API regressions**, after PR #1503/#1809 silently broke the `false` return of `on_pre_gui_draw_element` for nine days | REFramework issues/discussions, [alphazolam/EMV-Engine](https://github.com/alphazolam/EMV-Engine), REFramework Discord (web-indexed posts only) |
| The Evil Within | id Tech 5 | Any first public VR mod/injector attempt (none exists yet) | ModDB id Tech 5 page, Nexus Mods (Evil Within/Rage/Wolfenstein NOB-TOB), general search |
| Far Cry 2 | Dunia | Renderer architecture write-ups (e.g. the REAC 2023 Dunia shader-pipeline talk), any VR prior art | [enginearchitecture.realtimerendering.com](https://enginearchitecture.realtimerendering.com/) archive, Nexus Far Cry 2, general search |
| XIII (2003) | Unreal Engine 2 | UE2-era manual VR techniques (below UEVR's floor) | oldunreal.com, general UE2 modding forums |
| Unreal Gold | Unreal Engine 1 | 227k SDK updates, any new community render-device work | oldunreal.com, [OldUnreal GitHub org](https://github.com/OldUnreal) |
| Psychonauts | Double Fine bespoke (2005) | Any public camera/engine documentation; nothing turnkey exists. Our own generalised findings are on the [engine page](./engines/double-fine-psychonauts.md). | General search |
| DOOM (2016) | id Tech 6 | Whether the engine's **dormant inherited stereo path** is live or vestigial; any first public VR conversion attempt (none exists); Vk3DVision DOOM fix updates | [Vk3DVision fix list](https://3dsurroundgaming.com/Vk3DVisionGames.html), MTBS3D, Nexus (DOOM), general search |
| Manhunt (2003) | RenderWare | The family's one substantial VR conversion, **Vice City VR** — releases, and any write-up of how it does stereo. Watch for whether it ever grows an *injection*-side technique; today it is a source-port route that a game without a decompilation cannot take. `librw` itself for renderer documentation. | [vice-city-vr](https://github.com/dubrovskiy-yevhen-stakelogic/vice-city-vr) · [vice-city-vr-quest](https://github.com/Blackbird88/vice-city-vr-quest) · [librw](https://github.com/aap/librw) |

## Generic-driver ecosystem (fallback when not writing an adapter)

| Source | Why | Where |
|---|---|---|
| **Vk3DVision** (Helifax) | The only maintained **Vulkan** generic stereo driver; its per-game fix list is a fast feasibility check for Vulkan titles. Closed source — prior art only. | [github.com/helifax/Vk3DVision-Public](https://github.com/helifax/Vk3DVision-Public) · [fix list](https://3dsurroundgaming.com/Vk3DVisionGames.html) |
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
- **2026-08-24 (second sweep, later same day):** Checked all core tool/community sources for
  activity since the entry above. Added three genuinely new public items: **PCVR Central**
  (Camracks) — a new non-rehosting VR-mod discovery database, added to Community hubs and
  ATTRIBUTION; **Portal2VR** (Gistix/Portal4Dead, ~Dec 2025) — an injection-based Source-engine
  VR mod for Portal 2, added to `docs/source-available/` and ATTRIBUTION; **farcry_vrmod**
  (fholger) — a vendor-SDK (CryEngine Mod SDK) VR conversion of the original Far Cry (2004),
  added to `docs/source-available/`, `engines-index.md`, and ATTRIBUTION (not applicable to our
  Far Cry 2 project — different engine/game, no public SDK exists for Dunia — but useful
  landscape context, noted as such). Checked UEVR/REFramework/mutars/vrframework releases and
  Flat2VR Studios news: nothing new since the entry above (latest UEVR 1.05 and REFramework
  v1.5.9.1 predate this sweep; mutars repos unchanged). Noted but did not add: OldUnreal's
  v227k_15 (Aug 2024) is three releases ahead of the v227k_12 SDK the Unreal Gold VR project
  currently uses and includes fog/D3D9Drv rendering fixes relevant to that project's open
  fog-calibration work — this is project-specific SDK-upgrade intel, not general library
  material, so it's flagged here for the project's own next session rather than added to the
  library. A geo-11 "Mod Manager V3" mentioned on MTBS3D could not be verified (forum blocked
  automated fetch) — left out per the "leave it out if unsure" rule. Targeted checks for new VR
  prior art on The Evil Within (id Tech 5) and Far Cry 2 (Dunia) specifically: none found, per-project
  rows unchanged.
- **2026-08-26 (open-ended discovery sweep):** Checked core tool projects for commit/release
  activity since the 2026-08-24 entries above — nothing new: UEVR's last commit (2026-08-23)
  and REFramework's (2026-08-20) both predate that sweep; OldUnreal's v227k_15 is unchanged;
  elliotttate/vrframework last commit 2026-06-05; all of mutars' VR-relevant repos
  (starfield2vr, anvilengine2vr, Geo3D, stalker2-uevr) unchanged since their last-known dates.
  Checked REFramework's render-target/GPU-backing question (flagged in the Per-project relevance
  table as relevant to our RE Village work) directly — no new public information found, still
  open. Added one genuinely new item, verified across two independent sources (UploadVR,
  VR.org): Flat2VR Studios' parent, Impact Reality, opened a StartEngine equity-crowdfunding
  round ($5M+ already raised) and its campaign material revealed a much wider live/upcoming
  portfolio than the showcase alone showed, plus a separate PSVR2 publishing deal (Drop Dead:
  The Cabin, via sibling label Impact Inked) — added to docs/landscape/README.md and
  ATTRIBUTION.md. Flat2VR Studios' own news page showed nothing dated beyond 2026-08-24. PCVR
  Central's tracked-game count is essentially unchanged (~967 vs. the ~968 already on record) —
  no real signal either way. MTBS3D again returned HTTP 403 to automated fetch, same limitation
  noted in the prior sweep — still unverifiable this way, not confirmed unchanged.
- **2026-08-27:** Drained own inbox (empty — README only, nothing to fold in). Checked core tools:
  UEVR has one new commit today (D3D12 barrier-transition bug fix, no new release; still 1.05);
  REFramework unchanged since 2026-08-20; mutars repos, vrframework, OldUnreal (v227k_15) all
  unchanged since their last-known dates. Flat2VR Studios' news page shows nothing newer than
  2026-07-23; PCVR Central's count is unchanged (~967). Vk3DVision's DOOM (2016)/DOOM Eternal fix
  entries are unchanged since 2025-08-30/2024-12-30 — no new stereo-driver signal for the active
  DOOM project. Nothing added from the web this sweep — landscape is stable.
  **Project-repo harvest (by delta since 2026-08-26):** pulled all 17 `-engine-research` +
  16 `-external-research` clones; every one had only the routine PLAYBOOK-pointer/`inbox/`
  scaffolding commit except three with real content: **doom-2016-vr-engine-research** (Phase 0
  static+live recon, now fully covered — the id Tech 6 dormant-stereo findings were already folded
  into `docs/case-studies/id-tech-6-dormant-stereo.md` and `docs/engines/id-tech-6.md` in the prior
  sweep; this cycle's newer commits — Vulkan proxy M0 verified in-game, the launch-recipe/SteamAppId
  finding, the corrected injection-surface recommendation — are project-specific engineering detail
  except the SteamAppId mechanism, generalized below), **manhunt-2003-vr-engine-research** (read in
  full — windowed-mode D3D8 CreateDevice saga and the SecuROM packed-binary investigation; the
  packed-binary finding generalizes cleanly and is written up below, the windowed-mode D3D8 findings
  stay in the project dossier per the one-RenderWare-project rule), and
  **doom-2016-vr-external-research** / **manhunt-2003-vr-external-research** (research-session
  cross-checks, already folded into their own dossiers per the table above — nothing further to
  generalize from these two specifically). **Generalized up this sweep:** (1) a new case study,
  [`docs/case-studies/packed-binary-live-memory-scan.md`](./case-studies/packed-binary-live-memory-scan.md)
  — Manhunt's SecuROM investigation (16 documented addresses, zero static file matches, 16/16 live
  memory matches) generalized into "packed/self-protecting binaries: scan the live process, not the
  file," cross-linked from `docs/techniques/README.md` and a new **RenderWare** row in
  `docs/engines-index.md`; credited Fire-Head/MHNoDRM and the manhunt-2003-vr-* repos in
  `ATTRIBUTION.md`. (2) A new technique note in `docs/techniques/README.md`,
  "Launching a Steamworks game directly" — the `steam_appid.txt`/`SteamAppId` requirement (Valve's
  own Steamworks SDK mechanism) generalized from DOOM (2016)'s first-party confirmation (direct
  launch of `DOOMx64vk.exe` requires it) and cross-referenced against Far Cry 2's related-but-distinct
  Desktop-Game-Theatre wedge. Inboxes: none drained (all empty), none filled (no findings needed a
  project-specific hand-off this cycle).

### 2026-08-28 — inbox drain only (scoped by the user); no web sweep this run

**Scope:** the user asked for the inbox to be drained, not a full sweep. Web sources were **not**
checked this run — the watch-list bookmark is therefore UNCHANGED and the next full `/sr` should
still treat 2026-08-26 as its web-delta start. In-house harvest was likewise limited to the three
inbox files rather than a git-delta pass over every project.

**Inbox drained: 3 files, `flat-to-vr-cross-engine-research/inbox/` now empty.**

- `2026-08-27-mod-never-dispatch-engine-commands-from-render-hooks.md` (XIII modding session)
- `2026-08-28-mod-correction-xiii-gpf-was-not-the-render-path.md` — **superseded the above**
- `2026-08-28-gs-drain-these-two-together.md` — a `/gs` notice flagging the supersession

**This is the case the `Supersedes:` protocol was built for, and it worked.** The 08-27 file
claimed, as settled fact, that XIII's GPF proved "never dispatch engine commands from a
render-path hook". It was **disproved** the next day: re-arming that dispatch crashed the game
again from `ULevel::Tick` with no render path in the stack. Draining oldest-first and acting as I
went would have written the false claim into `docs/` and only then met its withdrawal. Both files
were still pending, so nothing wrong ever reached the library.

**Generalised up this sweep** — into `docs/techniques/README.md`, new section
["Driving a live game from a hook"](./techniques/README.md#driving-a-live-game-from-a-hook):

1. The dispatch-site advice **kept as guidance, with XIII explicitly relabelled as a case where
   the hypothesis was tested and FAILED** — not as its proof.
2. The narrower, better-supported finding: a global `Exec`-style entry point may be uncallable
   from an injected hook at all; prefer narrowly-scoped dispatch objects behind a default-off flag.
3. **Log before the call, flushed** — untouched by the correction and fully valid on its own.
4. The method lesson: *a fix that removes the symptom and its test coverage at the same time has
   proved nothing.* The most transferable item in the whole set.

Also grown: `docs/engines/unreal-1-3.md` gained its **first shared findings** (previously seeded
but empty) — the UE2 dispatch-object pattern with vtable-identity lookup, "standard UE command
therefore present" being false, alias-based input where useful aliases ship **unbound** (so a
binding, not code, is the fix), and two traps: a regenerated-on-launch ini, and an **unwrapped**
rotator that makes a shortest-arc wrap invent a direction reversal. `ATTRIBUTION.md` credits the
XIII repos, including the disproved diagnosis kept deliberately.

**Inboxes filled:** none. **Not drained (not this lane's to drain):**
`XIII2003-vr-external-research/inbox/2026-08-27-mod-f2-console-confirmed-and-cheat-map.md` (`/gr`)
and `doom-2016-vr-engine-research/inbox/2026-08-27-gr-devmode-enable-public-precedent.md` (DOOM's
modding session).

### 2026-08-31 — full sweep (web delta from 2026-08-27; in-house delta from 2026-08-27)

**Bookmark note for the next sweep:** the web-delta start is now **2026-08-31**. The 2026-08-28
entry above was inbox-only and correctly told the next sweep to use 2026-08-26; that gap is closed —
the 2026-08-27 entry's web pass plus this one cover it.

**⚠️ Estate layout changed under this library.** On 2026-08-30 the account consolidated 101 repos
into 22: each game is now ONE public repo `<prefix>` with folders `mod/ dev-archive/
modding-notes/ engine-research/ external-research/`, and the 79 old `<prefix>-<lane>` repos were
**deleted** (the old `-mod` repos were renamed, so those redirect; the deleted ones do not).
**Every in-account link in this library was therefore dead.** All 32 were rewritten this sweep to
the folder form (`…/<prefix>/tree/main/<lane>` and `…/<prefix>/blob/main/<lane>/ENGINE-DOSSIER.md`),
across `ATTRIBUTION.md`, both case studies, all eleven `docs/engines/` pages and
`docs/techniques/`. The engine pages' "All project repos" column is now "Project repo".

**Inbox drained: 4 files; `inbox/` is now empty.** No `Supersedes:` headers were present (checked
before draining, per the protocol). Two arrived from a `doom-2016-vr` modding session *during* this
sweep and were drained in the same pass.

- `2026-08-29-gr-re-engine-prefab-instantiate-pattern.md` → RE Engine family page
- `2026-08-29-gr-re-engine-family-anim-and-fire-origin.md` → RE Engine family page
- `2026-08-31-mod-measure-input-backends-against-a-control.md` → new techniques section
- `2026-08-31-mod-autocrlf-breaks-self-verifying-builds.md` → new techniques section

**Web sources checked.** *Two genuinely new items, both silent-failure bugs in tools we depend on:*

- **UEVR** — [PR #433](https://github.com/praydog/UEVR/pull/433) (Remleo, merged 2026-08-30): a
  gamma hook installed itself into a garbage vtable slot because an empty `std::optional` satisfied
  a `!= 0` test, so the "found it" branch ran precisely when it had not. Also a UESDK bump the same
  day. No new release; still 1.05.
- **REFramework** — [PR #1809](https://github.com/praydog/REFramework/pull/1809) (ErwinGunsmith,
  merged 2026-08-28): `on_pre_gui_draw_element` stopped honouring a `false` return as of
  [PR #1503](https://github.com/praydog/REFramework/pull/1503) (2026-08-19), so HUD-hiding scripts
  silently drew anyway for nine days. **Directly relevant to `visceral-re2-vr`, which shipped a
  crosshair-hiding script built on exactly that callback on 2026-08-30** — inbox drop filed.
- Unchanged: **mutars** (starfield2vr 2026-05-05, anvilengine2vr 2026-01-25, Geo3D, stalker2-uevr),
  **vrframework** (12 commits, no change), **OldUnreal** (still v227k_15), **Vk3DVision** (DOOM 2016
  and DOOM Eternal fix entries still 2025-08-30 / 2023-11-24 — no new stereo signal for the DOOM
  project), **Flat2VR Studios** news (nothing dated past 2026-07-23). **PCVR Central** is now **991**
  tracked games, up from ~967 four days ago — figure updated in `ATTRIBUTION.md`. **MTBS3D** was not
  re-attempted (403 to automated fetch in two prior sweeps).

**Project-repo harvest (delta since 2026-08-27, all 16 game repos pulled).** Eleven had only the
consolidation commits — Alan Wake, Alice: Madness Returns, arcade-controls, Burnout Paradise,
Enslaved, Far Cry 2, Mad Max, Prince of Persia, The Evil Within, Unreal Gold, and Manhunt (whose one
real commit was archival). Five had real content:

- **`psychonauts-vr`** — the black void **solved**: the camera transform the engine actually culls
  with, the three matrices that turned out to be derived outputs, the measured FOV-widen ceiling, and
  head-follow wired and monitor-validated. Dossier now **covered in full**.
- **`visceral-re2-vr`** — the aim-pose saga closed by pelvis-drop foot grounding, v0.1.0 shipped, and
  a full record of the animation-layer approaches that failed.
- **`re-village-scope-vr`** — the argument-encoding ABI root cause, the render-layer clipping kill,
  the layer control surface, and live exposure via tone mapping.
- **`doom-2016-vr`** — the developer-mode public-precedent-vs-first-party-reading tension (2026-08-27
  topic), plus the two mid-sweep inbox drops above.
- **`XIII2003-vr`** — dossier confidence-tagging and the corrected GPF diagnosis; already generalised
  in the 2026-08-28 entry, nothing further to lift.

**Generalised up this sweep.** Seven new sections in
[`docs/techniques/README.md`](./techniques/README.md): *the void behind the player* (with the
headset-free measurement method and the measured two-lever table), *finding the camera matrix the
engine actually reads*, *VR body height: the HMD-anchored float*, *silent no-ops: verification that
cannot see the failure* (the UEVR and REFramework bugs plus our own read-back-against-zero case),
*hook to acquire a handle the API will not give you*, *setting a gate before the process can guard
it*, *injected input: measure it against a control* (with a per-engine input-route table), and *tool
defaults that fabricate false negatives*. Two engine pages had their **first-ever shared findings**:
[`re-engine.md`](./engines/re-engine.md) (eight items) and
[`double-fine-psychonauts.md`](./engines/double-fine-psychonauts.md) (five). New credits in
`ATTRIBUTION.md`: alphazolam, Ekey, godlock2000-eng, Junh2x, Remleo, ErwinGunsmith, and prideslayer
(VRIK, cited only to distinguish the adjacent floor-calibration problem), plus first-party research
entries for the Psychonauts, Visceral and RE Village work.

**Two things worth flagging beyond the library.** (1) Every durable claim added this sweep carries a
confidence tag per the 2026-08-28 protocol, and one item is deliberately recorded as *disproved* —
the animation-layer attack on the aim pose, so a third RE Engine project does not walk it. (2) The
two independent public bugs found this week and our own ABI bug are the same shape, which is why
they were written up together rather than as three unrelated notes.

**Inboxes filled:** one — `visceral-re2-vr/external-research/inbox/` (the REFramework `false`-return
regression against their shipped crosshair script).

### 2026-09-01 — full sweep (web delta from 2026-08-31; in-house delta from 2026-08-31)

**The shape of this one:** the web was almost silent (one day since the last sweep), and the
in-house side was the richest single day this estate has had — **six projects across four engine
families all landed static-analysis wins on 2026-09-01**. Most of what follows was generalised out
of our own repos rather than found on the internet.

**Inbox drained: ten files, all of them.** `inbox/` is now empty but for its `README.md`. Five
`mod` drops from 2026-08-31 (id Tech 6 input measured; the attribution trap; verify the knob turns;
negatives need a positive control; write-combined reads and instrument validity), three `mod` drops
from 2026-09-01 (the no-op control for positives; console keys as layout-dependent dead keys, and
that file's own same-day correction), and two `gr` drops from 2026-09-01 (the stereo-separation
correction; three transferable heuristics).

**⚠️ The supersession scan earned its keep this time.** Running
`grep -rn "^Supersedes:" inbox/` before draining anything, per the 2026-08-28 claim-hygiene rule,
caught three claims that would otherwise have been curated as facts and only then met their
withdrawal:

- The *"per-frame ring buffers defeat address-based matrix hunting, proven by 319 vs 331
  survivors"* claim. Two later files in the same inbox withdraw that measurement — the "walk"
  condition never walked. The **mechanism** is still well-founded and is written up; the **number**
  is not cited, and the section says why.
- The earlier console-key file's specific VK values, superseded hours later by the discovery that
  the *active layout itself* changed between two launches on one machine. The corrected version is
  what was curated.
- A smaller one: a drop concluded *"the HUD is positioned from the view, so it follows the camera
  out of frame."* This session's own `/gr` research contradicts it — a developer-authored frame
  breakdown shows that game's UI is drawn to its own render target and composited last, so it cannot
  be culled by moving the world camera. The **rule** in that drop was curated; its HUD explanation
  was not.

**Web sources checked.** *Nothing new, as expected one day on.* **UEVR** and **REFramework** — zero
commits since 2026-08-31. **mutars** — most recent public activity 2026-08-22, unchanged.
**OldUnreal** — still v227k_15 (2026-08-16). **MTBS3D** not re-attempted (403 to automated fetch in
three prior sweeps). One genuinely new item, surfaced by this session's `/gr` rather than by the
watch-list walk: **Vk3DVision was archived by its owner on 2026-03-05**, read-only, final release
**4.25.5** — the fix list still shows DOOM (2016) at 2025-08-30, so the feasibility proof stands but
no future fixes will come. Recorded in the case study and `ATTRIBUTION.md`. Clarified alongside it:
the **6DoF** VR package from the same author exists **only for DOOM Eternal on id Tech 7**, built on
single-pass stereo instancing — so on id Tech 6 the public prior art is stereo-only, and the
long-open head-tracking question resolves to *no* for DOOM 2016.

**Project-repo harvest (delta since 2026-08-31; all 16 game repos pulled).** Seven had no
research-lane commits: Alan Wake, Alice: Madness Returns, arcade-controls, Burnout Paradise, Prince
of Persia, RE Village scope, Unreal Gold. Nine had real content, six of them substantial:

- **`mad-max-vr`** — **Denuvo blocks the executable, not the shader bundle.** 1363 DXBC shaders ship
  loose with `RDEF` reflection intact; `WorldViewProjMatrix` named at offset 0 of `InstanceConsts`.
  A project stalled at first injection became startable with no debugger. **Dossier covered in full.**
- **`enslaved-vr`** — the game ships its UE3 `.usf` sources; `c0`–`c3` is `ViewProjectionMatrix`,
  `c4` the camera position, `c5` `PreViewTranslation`. Settles a capture question by reading, and
  corrects an earlier reading that had counted *writes* rather than values. **Covered in full.**
- **`far-cry-2-vr`** — head tracking built and **verified numerically**; two composition bugs caught
  by a harness, one of which presents exactly as a handedness problem. **Covered in full.**
- **`XIII2003-vr`** — `FD3DRenderInterface::SetTransform` located statically as the single per-eye
  hook, **with an early-out on unchanged matrices that would collapse stereo silently.**
- **`the-evil-within-vr`** — the shader-layout coverage gap bounded to ten shapes, entirely off disk.
- **`doom-2016-vr`** — thirteen research-lane commits: the camera isolated to one static global, plus
  this session's own `/gr` topics. **Covered in full.**
- **`psychonauts-vr`**, **`manhunt-2003-vr`**, **`visceral-re2-vr`** — confidence-tagging passes and
  a windowed-mode registry finding; nothing further to lift that is not already generalised. Worth
  recording for its own sake: `visceral-re2-vr` **date-checked its pinned REFramework revision
  against the 2026-08-19→28 GUI-callback regression this sweep reported last time**, and confirmed it
  is not affected. That is the hand-off loop working end to end.

**Generalised up this sweep.** Eight new sections in
[`docs/techniques/README.md`](./techniques/README.md): *read the shipped files before you attach
anything* (with the DRM sub-lesson and the reflection-names-the-per-object-buffer-not-the-shared-one
limit), *counting events is not measuring content*, *stereo hazard: a setter that early-outs on an
unchanged matrix*, *composition bugs that masquerade as handedness*, *controls: a negative needs a
positive one, a positive needs a no-op one*, *never CPU-scan mapped GPU memory in place*, *driving a
game console with synthetic keys*, and *before you build it, check whether the game shipped it*.

The existing injected-input section was **substantially corrected**: the id Tech 6 row is now
measured rather than hypothesised, the **exclusivity** distinction replaces the API-family one, and
the "in-process is strictly stronger than SendInput" framing is withdrawn — that route installed
perfectly on id Tech 6 and moved the player zero metres. *Search by value, not by address* was added
to the camera-matrix section, and the large-file false-negative case to *tool defaults*.

**Four engine pages gained their first-ever shared findings** —
[`id-tech-6.md`](./engines/id-tech-6.md) (camera, stereo, injection, input),
[`avalanche.md`](./engines/avalanche.md), [`dunia.md`](./engines/dunia.md) and
[`id-tech-5.md`](./engines/id-tech-5.md) — and [`unreal-1-3.md`](./engines/unreal-1-3.md) gained a
camera-delivery section covering both UE3 and UE2.

The [id Tech 6 case study](./case-studies/id-tech-6-dormant-stereo.md) took its **first substantive
correction**: id's own GPL source says the view origin *is* adjusted per eye, which withdraws that
page's "override at the projection stage" recommendation in favour of the view stage. Its
doc-comment lesson was **sharpened rather than deleted** — *where an engine family has a published
ancestor, check the ancestor's source before building a plan on the descendant's comment.* The gate
section gained the community console-unlocker finding, and the prior-art section the archival status.

**Inboxes filled: three.** `doom-2016-vr/engine-research/` (try `+com_allowconsole 1` — the id Tech 5
gate name — as a launch-time probe against the known production gate);
`the-evil-within-vr/external-research/` (that game's console opens with `+com_allowconsole 1` and
`noclip` works, a live capability an entirely static project did not have on its board);
`far-cry-2-vr/external-research/` (`-DEVMODE` launch flag and a self-enumerating console, useful now
that head tracking needs a live check). All three came from asking whether DOOM's console-gate shape
recurs elsewhere in the estate — this sweep's own generalisation pointing back down at the projects.

**One thing deliberately not curated here.** `XIII2003-vr` recorded that its proxy source exists in
no repository — zero source files across 59 commits, the cited branch gone with the pre-consolidation
repos, and one unversioned copy on the home PC. `the-evil-within-vr` committed a recon log for the
same reason on the same day. That is estate hygiene rather than flat-to-VR knowledge, so it belongs
in `claude-memory`, not in this public library; flagged in the changelog line instead.

### 2026-09-01 (second sweep, later the same day) — the dossier-coverage backlog

**The shape of this one:** the previous entry was committed at 13:37 today, so both deltas were
tiny — the web had five hours to move and did not, and the only in-house commits since were the
three `inbox/` drops that sweep itself filed. Rather than report "nothing new" and stop, this pass
spent its budget on the **coverage bookmark**: five active projects whose dossiers had never been
read in full. That is where all the material below came from.

**Own inbox: empty** (README only). Nothing was waiting; `grep -rn "^Supersedes:" inbox/` returned
nothing, as expected on an empty inbox.

**Web sources checked — nothing new since 13:37, as expected.** **UEVR** newest commit still
2026-08-30 (the UESDK bump and the gamma-hook fix reported last time); still 1.05. **REFramework**
newest still 2026-08-28 (PR #1809). **mutars** — starfield2vr 2026-05-05, anvilengine2vr
2026-01-25, unchanged. **vrframework** last pushed 2026-06-09. **OldUnreal** Unreal-testing last
pushed 2026-08-16 (still v227k_15). **MTBS3D** not re-attempted (403 to automated fetch in four
prior sweeps).

**One genuinely new item, and it is the first VR prior art this library has found on the RenderWare
family.** [**Vice City VR**](https://github.com/dubrovskiy-yevhen-stakelogic/vice-city-vr) — an
unofficial stereoscopic 6DoF OpenXR conversion of the 2003 PC release of GTA: Vice City, publicly
active as of 2026-08-31, with a native-Quest sibling by **Blackbird88**. It was found by a targeted
query steered by the Manhunt dossier rather than by walking the watch list, which is the intended
order. **The method is the finding:** it is built on a reverse-engineered source reimplementation of
the game plus [**librw**](https://github.com/aap/librw) (aap, MIT) and **replaces the graphics
pipeline outright** — D3D12, single-pass stereo, VRS foveation, DLAA/FSR 2 — rather than hooking the
shipped renderer. So the existence proof is real and **its route does not transfer** to a
RenderWare title with no decompilation, which is every other game on this family here. Recorded on
the [RenderWare page](./engines/renderware.md), credited in `ATTRIBUTION.md`, and added to the
per-project relevance table as a new Manhunt row. Noted alongside it, because it affects what can be
planned: the underlying GTA III / Vice City reimplementation repository returns **HTTP 451 (access
blocked)** on GitHub as of today; `librw` is a separate, unaffected MIT project.

**Project-repo harvest (delta since 13:37 today; all 16 game repos pulled).** Thirteen had no
commits at all. The three that did — `doom-2016-vr`, `far-cry-2-vr`, `the-evil-within-vr` — carried
exactly the `inbox/` files the earlier sweep filed, so there was nothing new to lift by delta.

**Dossiers now covered in full (the bookmark for the next sweep).** Previously covered:
`psychonauts-vr`, `mad-max-vr`, `enslaved-vr`, `far-cry-2-vr`, `doom-2016-vr`, and
`manhunt-2003-vr` (recorded in `claude-memory`'s changelog on 2026-08-29 but never in this log —
noting it here so the two records agree). **This sweep adds four:** `the-evil-within-vr`,
`XIII2003-vr`, `visceral-re2-vr`, `re-village-scope-vr` — and re-read `manhunt-2003-vr` in full,
which is where most of the new material below came from. That is now every project `STATUS.md`
marks active. **Still never
read in full:** `unreal-gold-vr`, `alan-wake-vr`, `alice-madness-returns-vr`,
`burnout-paradise-vr`, `prince-of-persia-2008-vr`, `arcade-controls-re2-vr` (the last is frozen).

**Generalised up this sweep — nine new sections in
[`docs/techniques/README.md`](./techniques/README.md):**

- *Capturing the finished frame: the whole-frame route to a headset* (XIII) — the D3D8→D3D11-bridge
  mechanics, the three seam failures (overlay texture teardown, back buffer ≠ viewport, the readback
  cost) and, importantly, the **structural ceiling**: no stereo, no 6DoF, because the sim never sees
  the headset.
- *An old main loop may stop rendering the moment it loses focus* (XIII) — the `GetForegroundWindow`
  tick gate and the narrowly-scoped IAT fix, plus the zombie-process/single-instance-lock trap.
- *Identify a resource by how it is used, not by its creation descriptor* (The Evil Within) — with
  the two corollaries that cost that project two full rounds: check the resource's usage class before
  debugging why your read mechanism never fires, and a mechanism that runs perfectly can still be
  reading a decoy.
- *Deferred-context renderers: finding the world, and patching it once per eye* (The Evil Within) —
  including the domain-shader hazard, where a VS with no `SV_Position` is a category, not a bug.
- *The setting you want to change may be data, not code* (Manhunt) — the registry-driven video mode,
  the delete-the-key reset, and calling the engine's own enumerator instead of guessing an index.
- *Make one launch answer many questions* (Manhunt, The Evil Within) — the seven-variant probe sweep
  against a throwaway window that ended a three-session single-field hunt.
- *Remove your own code before accepting the blame — then fix the producer* (Manhunt) — the A/B with
  the proxy physically absent, and why patching consumers of a null is unbounded.
- *Prove the value you are debugging is the one the feature reads* (Visceral) — lookalike systems,
  parallel hierarchies, and measuring same-frame hook ordering with two probes instead of reasoning
  about it.
- *A flat game's "scope" is a fullscreen FOV zoom, and VR cannot use it* (RE Village) — with the
  measured 63° → 24.37° ramp, and the two probe techniques (recon flat-screen first; arm the probe,
  do not click it).

**Engine pages.** [`renderware.md`](./engines/renderware.md) gained its **first-ever shared
findings** (the prior art and what it does not transfer; the `SetTransform` lever; the protector
stub; DRM-remnant sabotage; the client-area raster check) and its identity block now says
fixed-function outright. [`id-tech-5.md`](./engines/id-tech-5.md) gained the deferred-renderer
section, the `K_eye` payoff of per-draw MVP delivery, the console/cvar asset, and the explicit
**no stereo heritage here, unlike id Tech 6**. [`unreal-1-3.md`](./engines/unreal-1-3.md) gained the
cheap-first-milestone section (why a proxy render device is idiomatic on this family, the exported
`eventPlayerCalcView` hook, the negated HMD yaw and the 65536-unit rotator) and its ceiling.
[`re-engine.md`](./engines/re-engine.md) gained scoped optics and the three-parallel-hierarchies
section.

**Inboxes filled: one.** `manhunt-2003-vr/external-research/` — the Vice City VR finding, written up
for that project as *what it proves and what it does not*, since Manhunt has no decompilation and the
D3D8 `SetTransform` route remains its only one.

**One judgement call worth recording, and it is the reason this pass was run at all.** Manhunt's
dossier was already marked covered, and the earlier sweep today saw its registry video-mode finding
and wrote "nothing further to lift that is not already generalised". Re-reading it in full changed
that: *the setting you want may be data, not code* is engine-agnostic, was nowhere in
the library, and is the kind of thing that saves ten live tests. **A delta scan reports what changed;
it does not tell you whether the unchanged part was ever harvested.** That is what the coverage
bookmark is for, and it is why this pass was worth running five hours after the last one.
