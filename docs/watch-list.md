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

> **Reading the entries below 2026-08-30:** they name repos as `<prefix>-engine-research`,
> `<prefix>-external-research` and so on, because that is what those repos were called on the day
> each entry was written. The [2026-08-30 consolidation](https://github.com/TefMeister/TefMeister)
> turned every one of them into a **folder inside one repo per game** — read
> `doom-2016-vr-engine-research` as `doom-2016-vr/engine-research/`, and so on throughout.
>
> ⚠️ **The old repos still exist as frozen duplicates** until the approved deletion pass, so those
> names still resolve — a reader who follows one lands in real-looking but stale content with no
> error to warn them, and the standing rule is **never push to those**. The historical entries are
> left as written rather than rewritten, because each correctly records the estate as it stood; this
> note is the fix. *(Raised by `/gs`, 2026-09-01.)*

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

### 2026-09-01 (third sweep, evening) — the coverage backlog cleared, and four withdrawn claims generalised

**Web: checked, effectively nothing to find.** The previous sweep ran at 14:22 today, so this was a
four-hour delta and treated as such rather than padded out. UEVR and REFramework were checked
directly for releases and for commits since 14:00: **zero commits on either, and the latest releases
are unchanged** — UEVR **1.05** (2024-11-16) and REFramework **v1.5.9.1** (2025-03-05), matching what
this log has recorded since 2026-08-24. No other watch-list source was worth re-querying on a
four-hour delta. Recording this explicitly because "nothing new" is a real result and the next sweep
should not re-check these two for a same-day delta.

**Inbox drained: six files, by explicit name** (`flat-to-vr-cross-engine-research/inbox/`) —
`2026-09-01-gr-executecommandlist-is-a-void-that-can-decline.md`,
`2026-09-01-gs-correction-the-changelog-split-was-08-28-not-09-01.md`,
`2026-09-01-gs-four-library-docs-use-off-vocabulary-tags.md`,
`2026-09-01-gs-id-tech-5-console-claim-is-undated.md`,
`2026-09-01-gs-sr-changelog-carve-out-points-at-the-wrong-file.md`,
`2026-09-01-gs-watch-list-names-retired-repos.md`. The whole inbox was read before any of it was
folded in, and `grep "^Supersedes:"` run first — which mattered: the correction drop withdraws the
provenance of the changelog-split drop (the split was `0dae26d` on **2026-08-28**, not `03daa1d` on
2026-09-01), and draining oldest-first would have written the wrong date into this log.

**Tag hygiene fixed (`/gs` check 3b).** Five off-vocabulary confidence tags, four of them a hyphen
away from valid: `docs/case-studies/id-tech-6-dormant-stereo.md` and `docs/techniques/README.md` ×2
and `docs/engines/dunia.md` now use `[reported]` / `[verified-live … n=]` /
`[verified-numerically …]`, with the precision moved into prose. `docs/engines/id-tech-5.md`'s bare
`[verified-live]` on the console/cvar claim is now `[verified-live 2026-08-21, n=1 game]` and says
outright that the family here **is** one game (The Evil Within), with any other id Tech 5 title
`[hypothesis]`.

**The `/sr` changelog carve-out needs nothing from me** — it was fixed by the modding lane on
2026-09-01 and this session's command file already names `STATUS-CHANGELOG.md`.

**Retired repo names in this file: fixed by a note, not by rewriting history.** `/gs` flagged five
pre-consolidation repo names here. All five are inside **dated sweep-log entries**, which is the
category `/gs`'s own scope note says not to rewrite — each correctly records the estate as it stood.
Rewriting them would falsify the record; leaving them lets a reader land in a frozen duplicate with
no error. **Both problems are solved by one note at the top of this log** explaining the folder
mapping and warning never to push to the old repos. Recorded here because it is a judgement call the
next sweep may want to revisit.

**Project-repo harvest (delta since 14:22 today; all 16 game repos pulled).** Fourteen had no commits
at all. `the-evil-within-vr` carried only a `/gs` inbox drop. **`doom-2016-vr` had six commits and
was the whole harvest** — an engine-research inbox drain and an ASLR correction, both substantive.

**Dossier-coverage backlog: CLEARED.** Every project's `ENGINE-DOSSIER.md` has now been read in full
at least once. This sweep covered the last six — `unreal-gold-vr`, `alan-wake-vr`,
`alice-madness-returns-vr`, `burnout-paradise-vr`, `prince-of-persia-2008-vr` and
`arcade-controls-re2-vr` — which together are smaller than DOOM's dossier alone (~640 lines) and had
been deferred as low-priority because the projects are early or paused. That was the wrong reason:
**early-stage dossiers are where the recon and first-injection lessons are**, and four of the five
new library sections below came out of them. Future sweeps revert to delta-only, except where a
dossier grows substantially.

**Generalised up this sweep — five new sections in
[`docs/techniques/README.md`](./techniques/README.md), plus one new guard:**

- *The switch you cannot find may be an argument, not a global* (DOOM) — the two independent negative
  reads (6,572-cvar published dump; 171 registered at runtime), `RB_DrawView(data, stereoEye)` and
  `viewEyeBuffer` from id's published GPL source, and the diagnostic that transfers: **every
  parameter of a feature exposed while nothing selects the mode is the signature of a call-site
  argument.** Carries the present-vs-registered distinction explicitly, because that is the trap next
  door.
- *A repeated launch is not an ASLR test* (DOOM) — three same-base launches then a fourth, no reboot,
  at a different base. The transferable half is not about ASLR: **trials inside one cache-warm window
  are one trial repeated**, so `n=3` counted as `n=1`. Plus the reassuring corollary — the
  `GetModuleHandle(NULL) + RVA` procedure was immune to the question either way.
- *A third-party stereo fix is free intelligence about the engine* (Alice, Alan Wake, Prince of
  Persia, Burnout — `n=4`) — **the best find of the sweep.** Three inference routes: what the fix did
  *not* fix tells you the native camera is already right; its issue list is a free pass inventory;
  its config structure encodes engine structure (separate cutscene/gameplay convergence presets mean
  two camera paths). With the Burnout negative — a fix for the D3D9 original and none for the D3D11
  remaster — as the caveat that build and renderer must match, not the title.
- *A proxy DLL must export everything the target actually imports* (Alice, Alan Wake, Prince of
  Persia — `n=3`) — Alice statically imports **two** `d3d9.dll` functions, so a one-export proxy died
  before `main` with no log at all; Prince of Persia needed one and worked first time. The table of
  **static-vs-dynamic failure modes** is the part that transfers: total silence means the export
  table, a tidy error message means your logic.
- *The instrument can be the bug* (Alan Wake, `n=1`) — a `CreateDevice` vtable hook added to diagnose
  a crash *was* the crash, and a Fault-Tolerant Heap shim took credit for a fix it never performed.
  Also the habit of keeping a known-bad instrument disabled with a note.
- **New guard 7** under *Controls* (arcade-controls-re2-vr) — **a signal must be able to separate the
  states before you tune a threshold on it**: camera-to-head distance measured 0.111 m at rest
  against 0.112 m during an enemy grab, so no threshold could ever have worked. Distinct from guard
  5, where the two conditions were accidentally identical rather than genuinely indistinguishable.

**Engine pages.** [`id-tech-6.md`](./engines/id-tech-6.md) gained the call-argument stereo switch with
its consequence that **the console gate is off the critical path for stereo**, and a correction: the
`explicit*` override fields are **not cvars at all**, so `rp` or a patch is their route.
[`remedy-alan-wake.md`](./engines/remedy-alan-wake.md) and
[`scimitar-anvil.md`](./engines/scimitar-anvil.md) were **populated for the first time** — both had
sat as empty seed pages since 2026-08-26. Remedy's covers the thin-loader/per-subsystem-DLL
architecture (which changes how you do recon on it), dynamic D3D9 resolution, the shipped
`-freecamera` / `-developermenu` tools, reported native 3D Vision, and the vtable-hook warning.
Scimitar's covers the confirmed AC1 codebase lineage, `.forge`, D3D9-not-Ex, the embedded default
command line with its `/noconsole` tell, and the two-camera-path inference. `id-tech-5.md` and
`engines-index.md` took the tag and stereo-switch corrections.

**Case study.** [`id-tech-6-dormant-stereo.md`](./case-studies/id-tech-6-dormant-stereo.md) gained a
dated update section: its three lessons all stand, but **the prize behind the gate is smaller than it
looked** — winning the console yields stereo parameters, not the on-switch. A fourth lesson was added:
*when a switch cannot be found, question the category before questioning the search.*

**Inboxes filled: two.**
`doom-2016-vr/engine-research/inbox/` — **§13's number-one next step calls `multiView_60Hz`
"registered, ungated" while §9 lists it as never registered.** §12 read it from the published dump
(the binary's inventory); §9 measured what retail registers. Those are different axes and §13
conflated them, so the top item's stated price — "needs no gate work at all" — may be wrong by the
width of that project's largest open blocker. One `listCvars multiView` in any future session settles
it. Flagged item 3 in the same list too, which still says a reboot is needed to test rebasing after
the same evening's commit disproved exactly that.
`alice-madness-returns-vr/engine-research/inbox/` — **§6's `c0` caution is superseded.** That dossier
tells the next session to expect the hard per-object-WVP case, on a sibling project's reading that
has since been corrected: `c0`–`c3` **is** the shared `ViewProjectionMatrix`, `c4` hands over the
camera position, and the 47-uploads-per-frame observation was UE3's RHI re-applying reserved
registers. Includes the clean injection point, the `PreViewTranslation` drift trap, and an honest
statement that this is `[inferred-static, n=1]` from Enslaved's shaders and needs verifying on
Alice's own binary.

**Attribution.** Microsoft's D3D11 reference (the `ExecuteCommandList` hazard) and the HelixMod
community including Chiz (per-game fix write-ups, **read online only, never installed or copied**)
were added, alongside the four projects behind the new sections.

**One thing worth carrying forward.** Four of the six new entries came from claims their own projects
had **withdrawn** — the ASLR reasoning, the "find the mode cvar live" advice, the FTH shim, the `c0`
reading. The corrections were the transferable content, not the original findings, and none of them
would have surfaced from a delta scan of changed files: three were sitting in unchanged dossiers that
had simply never been read here. That is the case for clearing a coverage backlog even when every
repo reports no commits.

**Addendum, same sweep — a seventh inbox file arrived mid-drain, and the drain rule caught it.** While
this sweep's commit was being prepared, a `/gr` session pushed
`inbox/2026-09-01-gr-clip-space-stereo-footer.md` at **18:56**, after this drain had begun. Because
the six drained files were deleted **by explicit name** rather than by glob, the new drop survived
untouched — the first observed instance of the concurrent-drop window the
`CONVENTIONS.md` rule was written for, and it behaved exactly as designed. It was then folded in
during the same session rather than left for the next sweep, since it bore directly on work this
sweep had just published.

**What it added — a new section, [the clip-space stereo
footer](./techniques/#the-clip-space-stereo-footer-geometry-stereo-without-ever-finding-the-camera).**
NVIDIA's own documentation for 3D Vision Automatic describes the mechanism behind the entire
geo-11 / HelixMod / 3Dmigoto ecosystem this library already catalogued as drivers-you-point-at-a-game:
a footer appended to every vertex shader, `ClipPos.x += Separation * (ClipPos.w - Convergence)`, with
each draw issued twice. **It yields real geometry stereo while knowing nothing about the camera,
view matrix, projection or handedness** — a second route to two eyes alongside this library's
camera-hunting material, implementable by our own proxies with no NVIDIA driver or GPU. Its
documented costs (per-game draw-call exclusion, broken unprojection in deferred/post-processed
renderers) are recorded with it, as is a cross-link both ways with `docs/generic-drivers/`.

**⚠️ And it corrected a claim this sweep had published an hour earlier.** The new
*third-party stereo fix* section had read Alan Wake's `Ctrl+F3`/`Ctrl+F4` separation hotkeys as
evidence of "a per-eye offset mechanism already wired up". **Those are the *driver's* hotkeys in 3D
Vision Automatic mode**, where the driver splits the draws and owns the parameters — so they show 3D
Vision working *on* the game, not a native per-eye path in it. Corrected in both
`docs/techniques/README.md` and `docs/engines/remedy-alan-wake.md`, with the static check that
settles it (which driver mode the binary requests via `NvAPI_Stereo_SetDriverMode`). **Alice's signal
survives and Alan Wake's weakens**, because Alice's rests on the fix author's statement about the
*game*, not on driver-side controls. Recorded rather than quietly edited, since the generalisation
was published before the correction arrived.

### 2026-09-02 — full sweep (web delta from the third 2026-09-01 sweep; in-house delta ~4.5h)

**Web: two direct source checks, both aimed at closing an existing tag rather than open-ended
browsing.** UEVR/REFramework were not re-checked — the prior entry already recorded them stable on a
same-day delta and nothing in this window's harvest touched them. Two targeted fetches instead:
NVIDIA's `nvapi_interface.h` (confirmed all six NVAPI stereo/init function IDs the Alan Wake project
needed named, including `NvAPI_Stereo_SetDriverMode` = `0x5E8F0BEC`) and NVIDIA's `nvstereo.h`
documentation page (confirmed the `StereoParmsTexture` channel layout already recorded by the Alice
project's own research — no new information there, but now independently verified rather than
single-sourced). A search-based check of Epic's UDK "3D Vision Direct" page corroborated the ini key
and fullscreen-only restriction the same way the project's own inbox drop had it (page itself still
403s to fetch).

**Project-repo harvest (delta since 19:25 on 2026-09-01; all 16 game repos pulled).** Twelve had real
research commits — the heaviest single-day in-house delta this log has recorded. Fourteen dossiers
were touched by the modding lane folding in the previous sweep's own inbox drops (a visible feedback
loop: this account's sweeps are now a measurable fraction of what the modding lane has to process).
Read in full: `alan-wake-vr` (NVAPI caller-count method + its own claim-hygiene correction),
`alice-madness-returns-vr` (CTAB register split, `NvStereoFixTexture` layout), `doom-2016-vr`
(framespy measurement, linear-allocator uniform buffer, the resubmission-is-legal finding, and a
retracted patching conclusion), `enslaved-vr` (D3D9-not-Ex, the shared-handle bridge and its two
traps), `XIII2003-vr` (UE2 doc/NDA split), `psychonauts-vr` (Astralathe), `prince-of-persia-2008-vr`
(`.forge` container vs. datablock-schema split), `mad-max-vr` (2024 geo-11 fix supersedes 2015 prior
art), `manhunt-2003-vr` (RenderWare VR prior art), `visceral-re2-vr` (the FirstPerson lerp-vs-equality
bug, read from REFramework's own published source), `burnout-paradise-vr` and `far-cry-2-vr` (both
routine check-ins, no new engine content).

**Generalised up this sweep — four new sections in
[`docs/techniques/README.md`](./techniques/README.md), plus two in-place extensions:**

- [Counting callers separates what a binary links from what it uses](./techniques/README.md#counting-callers-separates-what-a-binary-links-from-what-it-uses)
  (Alan Wake, `n=1` game + `n=1` reproduced-here method) — the NVAPI-ID caller-counting technique that
  decided a whole section of that dossier without a launch, plus the claim-hygiene lesson: the project
  correctly split "structure verified" from "naming unverified" rather than letting one lend
  confidence to the other, and this sweep closed the naming gap from NVIDIA's own published table. A
  `/gr` session independently generalised the same finding into that project's own dossier on the same
  day (worth recording: two sessions reached the same generalisation from the same raw finding within
  hours, and this sweep's inbox drop into `alan-wake-vr` answering the open `/pd` request arrived
  minutes after `/gr` had already answered it — redundant but harmless, left for `/gr`'s next drain to
  recognise and delete rather than touched further).
- [Both eyes from one recorded frame: resubmitting the game's own command buffers](./techniques/README.md#both-eyes-from-one-recorded-frame-resubmitting-the-games-own-command-buffers)
  (DOOM, `n=1`) — legal command-buffer resubmission as a fourth stereo-submission strategy on explicit
  APIs, with the linear-allocator trap that invalidated an older patching conclusion.
- [D3D9 to a modern VR compositor: the shared-handle bridge](./techniques/README.md#d3d9-to-a-modern-vr-compositor-the-shared-handle-bridge-and-its-two-traps)
  (Far Cry 2 → Enslaved, `n=2` projects) — the documented D3D9Ex/D3D11 shared-texture interop path,
  the three-way static check for whether a game has a D3D9Ex device at all, and the `D3DPOOL_MANAGED`
  / no-keyed-mutex traps that decide the design.
- [Never gate a state change on exact equality with a lerp target](./techniques/README.md#never-gate-a-state-change-on-exact-equality-with-a-value-that-only-lerps-toward-its-target)
  (Visceral RE2, `n=1`, from published upstream source) — the general form of "test the state, don't
  infer it from a value merely converging on it," plus the tempting workaround that provably does
  nothing.
- **Stereo-parameters-texture lever**, folded into the existing dormant-native-stereo section — Alice's
  find that `NvStereoFixTexture` is NVIDIA's documented, application-provided `StereoParmsTexture`
  turns the section's "the app must patch its shaders" caveat into a lever a proxy can drive directly,
  with the Direct-vs-Automatic tension stated honestly rather than resolved past the evidence.
- **Two additions to "a third-party stereo fix is free intelligence"** (Mad Max, Prince of Persia) —
  read the fix's own files, not its announcement, for register-level detail; and check that recorded
  prior art still names the *current* fix, not a nine-year-old one a newer author has since superseded.
- **One addition to "check whether the game shipped it"** (Psychonauts) — check whether the *community*
  already built it, and specifically whether a discovered tool collides with your own proxy before
  installing it.

**Engine page. [`unreal-1-3.md`](./engines/unreal-1-3.md) had its heaviest single-sweep growth since
seeding:** the UE3/D3D9 register map is now `n=2` from two different evidence types (Enslaved's
shipped `.usf` source vs. Alice's compiled shader reflection) with the vertex/pixel register-space
split recorded as a trap that nearly produced the wrong reading; a new section records that UE3
shipped an **official Epic/NVIDIA 3D Vision Direct integration** behind one ini key, stated alongside
— not resolved against — the competing Automatic-pattern evidence found in the same engine generation's
shaders; and a new UE2 section separates "the documentation is public" from "the headers were always
NDA-gated," with Epic's own page corroborating that the render-device layer this family's projects
already converge on is the historically sanctioned VR-driver seam. [`scimitar-anvil.md`](./engines/scimitar-anvil.md)'s
`.forge` bullet was rewritten to state plainly that public knowledge of the format stops at assets —
no compression scheme, type IDs, or datablock schema exists publicly for this engine generation at all
— while recording the `scimitar` header identifier as a free engine-identity check and cross-linking
to the search-by-value technique that makes the schema gap not matter for the current critical path.

**Attribution.** NVIDIA (the two published headers), Epic (the UDK 3D Vision and UE2 RuntimeHeaders
pages), DHR/Rubini and the 3D-fix archive community, the `broadside` wiki, Turfster/AnvilToolkit, and
Astralathe's/PsychonautsStudio's authors were added, credited by name for the specific claims drawn
from their work.

**Inboxes: drained three from my own, across two waves** — the first two (both duplicates/refinements
of the same NVAPI-verification finding) arrived in the concurrent-drop window this log has now
observed twice, and a third landed *after* the first drain and after this sweep's own commit had
already been rebased onto it: `nvstereo.h`'s texture is app-written, not driver-written — a genuine
correction to this sweep's own just-published wording, since the earlier NVIDIA doc page's
"app-provided" language had been (mis)read here as "driver-filled" — plus its exact 8×1
`A32B32G32R32F` shape and `NVSTEREO_IMAGE_SIGNATURE`, now folded into the stereo-parameters-texture
section with the correction stated rather than silently fixed. All three survived intact because
deletion was by explicit name each time. **Filled one** — `alan-wake-vr/external-research/inbox/` got
this sweep's independent NVAPI-table confirmation, which turned out to race a `/gr` session that
answered the identical `/pd` request first; recorded above rather than corrected, since both answers
agree and no false claim resulted.

### 2026-09-02 (second sweep, afternoon, dev PC) — a short in-house delta, and the UE1–3 page grows a UE1 half

**Web: the watch-list's core sources are all unchanged.** UEVR (last release 1.05, 2024-11-16),
REFramework (v1.5.9.1, 2025-03-05; its ten most recently updated issues carry nothing on the Lua GUI
callback, render targets or scene layers — the one VR-tagged issue is PSVR2 Sense-controller input on
RE9), OldUnreal `Unreal-testing` (v227k_15, 2024-08-16), dgVoodoo2 (v2.87.3, 2025-06-23), mutars
(starfield2vr v2.0.1.Public 2026-05-05, anvilengine2vr v2.0.0.Public 2026-01-25) and vorpX (25.1.5,
January 2026) all show no release since the last sweep. PCVR Central's library page no longer prints its
catalogue counts, so the 2026-08-31 figure stands as the last recorded. One landscape gap was found and
filled rather than anything new: **Flat2VR Spark** (announced August 2025 — modders building licensed
adaptations from source, in pods, with revenue participation) was absent from
[`docs/landscape/README.md`](./landscape/README.md) despite several of its slate appearing in the
portfolio list already there; added with UploadVR's report as the source. A steered search for which
UE3 pixel shaders consume the view-projection (Enslaved's new open risk) found nothing public worth
recording.

**Project-repo harvest (delta since 03:06 on 2026-09-02, the previous sweep's commit; all 16 game repos
plus `staging` pulled).** Four repos had commits in the window and only two carried research content —
the rest were `/gs` inbox drains and tag corrections. But two commits from **02:54–02:55** on
`enslaved-vr` and `unreal-gold-vr` landed inside the previous sweep's window *after* it had pulled, and
`unreal-gold-vr` was not in that sweep's list at all, so both were harvested here as new. Read in full:
`enslaved-vr` (the stage-qualified CTAB re-walk and the NVIDIA-branch negative), `unreal-gold-vr` (the
M2 stereo proof note, its dossier delta, and the two `/gr` topics from 2026-09-01 22:31 on the
script-event camera and the ICBINDx11Drv provenance), `XIII2003-vr` (dossier: the rescued-tree
read-only rule), `psychonauts-vr` (modding notes 71 and 72 — the second superseding the first),
`alice-madness-returns-vr` (the `NvStereoFixTexture` format topic — already folded by the previous sweep's
correction commit, confirmed, nothing further). Twelve repos unchanged since the bookmark.

**Generalised up this sweep — two new sections and three in-place extensions in
[`docs/techniques/README.md`](./techniques/README.md):**

- [Prove the test can fail: mutation-check a numerical verification](./techniques/README.md#prove-the-test-can-fail-mutation-check-a-numerical-verification-before-trusting-it)
  (Unreal Gold, `n=1`) — the independent-ground-truth / different-precision / mutation-run method that
  verified per-eye stereo maths without a launch, framed as controls rule 1 applied to a passing test,
  plus the write-the-launch-outcome-table-first habit and the no-convergence design note.
- [When byte-identity is the evidence, the tree is read-only](./techniques/README.md#when-byte-identity-is-the-evidence-the-tree-is-read-only)
  (XIII, `n=1`) — fix the scanner that flags a rescued tree, never the file; branch, never edit in place.
- **Controls rule 1 gains a fourth worked failure shape** (Psychonauts, `[disproved 2026-09-02]`) — a
  flawlessly delivered stimulus the system was never bound to respond to, the positive control that was
  already in the repo, and "three parameter sets of one API are not three routes"; with a matching
  "read the game's bindings first" paragraph in the injected-input section and a corrected Psychonauts
  row in the input-routes table.
- **Eye height** (Psychonauts, `[measured 2026-09-02]`) — a new subsection under the body-height section:
  camera-minus-player is a third-person camera height, take it from the head bone instead, and check the
  ground is level.
- **HUD & UI** (Unreal Gold) — keep the 2D layer full-window mono during a flat SBS proof, because the
  mouse maps to the window.

**Engine page. [`unreal-1-3.md`](./engines/unreal-1-3.md) — the UE3 register trap is now `n=2` and the
page finally has a UE1 half.** The 2026-09-01 "split reflection tables by shader target" warning, written
from Alice's near-miss, is now backed by Enslaved's actual mis-staged "9% at `c3`/`c10`" claim and its
correction: on the vertex side the view-projection is at `c0` and nowhere else, the pixel-side slot
differs per title (ps `c3` Enslaved, ps `c4` Alice), and the un-offset pixel-side view-projection is a
recorded open risk. The 3D Vision section records that the NVIDIA integration is **per licensee build,
not per engine generation** (Alice has it in 65% of pixel shaders; Enslaved has zero occurrences across
eight files) and that the right check is a byte grep over the caches, which also covers the SM4 cache a
`CTAB` walker cannot read. New UE1 material: no view matrix, so the per-eye camera is a view-space
translation carried by one extra constant (numerically verified, not yet rendered); and **head-look
needs no native code on UE1 or UE2** because the view is produced by a script event — the family's
camera work splits cleanly into orientation at the script layer and per-eye offset/projection at the
render device, with the pose bridge the one native piece (documented community procedure on UE1, 64-bit
unverified). The engines index still lacks a UE1 row; added this sweep.

**Attribution.** The Unreal Wiki (via Unreal Archive), OldUnreal's public 227 UnrealScript, the two
OldUnreal forum authors of the native-package procedure, the BeyondUnreal wiki, and Flat2VR Spark
(via UploadVR) were added; the first-party paragraph now names `unreal-gold-vr` and the specific
Enslaved, XIII and Psychonauts findings generalised here.

**Inboxes: drained none** — my own held only its `README.md` (the `/gs` drop from 09:54 had already
been folded at 10:20). **Filled one**: `unreal-gold-vr/external-research/inbox/` got the public
Unreal-unit scale evidence that bounds the M2 `StereoIPD=3.4` hypothesis and shows the documentation
cannot settle it — three public figures (52.5, 50 and 44.6 UU/m) put 3.4 UU anywhere from 65 to 76 mm,
so the dossier's plan to measure in the headset is the only route. No `engine-research/inbox/` drops
this sweep; nothing answered a dossier dead end directly.

### 2026-09-02 (third sweep, evening) — a `/gr` estate sweep in one afternoon, harvested whole

**The shape of this one:** a single `/gr` estate sweep ran across the whole account between the
previous entry's bookmark (03:06) and now, touching sixteen of sixteen game repos with either a
CHECK-IN or FULL pass and landing two same-day cross-project verifications. This session's own inbox
also received three genuinely new drops mid-sweep. Almost everything below came from reading that
work rather than from the open web.

**Own inbox drained: three files, by explicit name** (`grep -rni "supersedes" inbox/` returned
nothing first) —
`2026-09-02-gr-gitlab-rest-api-reads-what-the-web-ui-hides.md`,
`2026-09-02-gr-when-a-game-compiles-its-shaders-decides-how-you-read-its-constant-map.md`,
`2026-09-02-mod-compressor-enum-crc32-types-and-a-positive-control-that-earned-its-keep.md`. All
three were genuinely new generalisations, not corrections, and all three are folded into the library
(below); `inbox/` is now empty but for `README.md`.

**Web checked, unchanged as every same-day check today has found:** UEVR (still **1.05**,
2024-11-16, checked directly against the releases page) and REFramework (still **v1.5.9.1**,
2025-03-05). No other watch-list source was queried on top of the four direct checks already
recorded earlier today.

**Project-repo harvest (delta since 03:06; all 22 repos in this root pulled, 0 changes on any of
them — every commit below was already on `origin` from the concurrent `/gr` sweep).** Research-lane
commits found on: `XIII2003-vr`, `alan-wake-vr`, `alice-madness-returns-vr`, `burnout-paradise-vr`,
`doom-2016-vr`, `enslaved-vr`, `far-cry-2-vr`, `mad-max-vr`, `manhunt-2003-vr`,
`prince-of-persia-2008-vr`, `psychonauts-vr`, `re-village-scope-vr`, `the-evil-within-vr`,
`unreal-gold-vr`, `visceral-re2-vr` — fourteen of sixteen, the heaviest single-day breadth this log
has recorded (only `arcade-controls-re2-vr`, frozen, had nothing). Most were routine `/gr` rotation
CHECK-INs; the substantial ones are generalised below.

**Generalised up this sweep — three new sections, three in-place extensions, in
[`docs/techniques/README.md`](./techniques/README.md):**

- [**OpenXR carries a pose per view where OpenVR collapses to
  one**](./techniques/README.md#openxr-carries-a-pose-per-view-where-openvr-collapses-to-one)
  (`far-cry-2-vr` → `XIII2003-vr`, same day, `[verified-static 2026-09-02]`) — the best cross-project
  moment of the day: `far-cry-2-vr` re-checked the seven-year-old, still-open
  [OpenVR #1253](https://github.com/ValveSoftware/openvr/issues/1253) (filed by **LukeRoss00**, the
  AER author, describing exactly this same-frame-pose collision) and reasoned that OpenXR's
  projection layer *should* avoid it by carrying a pose per view — tagged `[hypothesis]`, flagged for
  verification. An hour later, on a sibling project, that verification happened against Khronos's own
  published header: `XrCompositionLayerProjection` really does hold an array of per-view poses in one
  layer/one space, so the last-submit-wins collision structurally cannot occur. The same pass also
  caught a layer-type trap worth keeping: a **quad** layer (the common M1 "flat panel" host) has one
  pose and cannot carry stereo at all — only the **projection** layer can. Runtime honouring stays
  untested and is one cheap headset check away.
- [**When a game compiles its shaders decides how you read its constant
  map**](./techniques/README.md#when-a-game-compiles-its-shaders-decides-how-you-read-its-constant-map)
  — folded from the drained `mod` inbox file (three projects converging on the same question: source
  shipped → read it; compiled-and-stripped → parse `CTAB`/`RDEF`; compiled at runtime → hook the
  compiler). **Corroborated the same afternoon** by `alan-wake-vr`'s own `/gr` pass, which found the
  game ships `d3dcompiler_42`/`43` redistributables and fails to launch without them — direct
  first-party evidence for the third row, now folded in with the "proxy seam" framing and credit to
  the community troubleshooting thread that documents the failure string.
- [**The executable can name its own compressed formats and type
  hashes**](./techniques/README.md#the-executable-can-name-its-own-compressed-formats-and-type-hashes)
  — folded from the drained `mod` inbox file (`prince-of-persia-2008-vr`'s completed `.forge` decode,
  `[verified-numerically 2026-09-02, 7.90 GB reproduced]`): grep the strings table for a compression
  library's own enum names before guessing a variant from byte patterns, and check `crc32(name)`
  against any stored 32-bit value beside a name before assuming a custom hash — 201 of 202 type
  hashes resolved this way in one pass. Credited **LZO** (Markus F.X.J. Oberhumer) for the decoder
  constants transcribed to verify it.
- **A fifth shape added to [controls rule
  1](./techniques/README.md#controls-a-negative-needs-a-positive-one-a-positive-needs-a-no-op-one)**
  — the same Prince of Persia session's positive control: three certainly-present state hashes were
  searched for and found only in audio data, proving the target data stores states as ordinals, not
  hashes, *before* a null on the actual target state got misread as "stripped from the shipping
  build." Third instance of this exact pattern in three days (Psychonauts, this one, and see below) —
  worth naming as a recurring habit, not a coincidence.
- **A new item folded into [tool defaults that fabricate false
  negatives](./techniques/README.md#tool-defaults-that-fabricate-false-negatives)** — the drained `gr`
  inbox file's GitLab finding: a client-side-rendered project/wiki page returns only a loading shell
  to an automated fetch, indistinguishable from genuinely empty, where the site's REST API (no token
  needed for public projects) returns the real tree/file/wiki content underneath. Found on
  `psychonauts-vr`'s own research toolbox, applied by `/gr` to read Astralathe's GitLab-hosted source.
- **A caveat added to the [Automatic-vs-Direct
  diagnostic](./techniques/README.md#-the-diagnostic-that-matters-for-recon-automatic-vs-direct)**,
  plus a **new entropy-signature paragraph in [packed/self-protecting
  binaries](./techniques/README.md#packedself-protecting-binaries)** — both from `alice-madness-returns-vr`'s
  `/pd` session, `[measured 2026-09-02]`: the exe's `.text` reads at entropy 8.00 with its entry point
  outside `.text` and zero `CC` padding runs (a wrapper signature readable from the PE headers alone,
  no disassembly needed), which makes the caller-count static scan for NVAPI driver mode a guaranteed
  false negative rather than a real answer — re-gating that `[PD]` item to `[FLAT]`. The **third**
  instance of the positive-control pattern this sweep touched: the readable `.rdata` NVAPI interface
  table looked like it settled Direct-vs-Automatic by omission, until checking whether
  `NvAPI_Initialize` itself — certainly called — was also absent (it was), proving the table is the
  linked SDK's fixed list, not the game's usage.

**Not generalised, deliberately.** `enslaved-vr`'s D3D9Ex/`MANAGED`-pool sidestep (the shipped D3D10
path avoids the trap this library already documents) and `manhunt-2003-vr`'s `MHWSF`-sourced camera
globals are real findings but stay project-specific — the first is a single project's design choice
around an already-documented general trap, the second is RenderWare address-level detail with no
second data point yet.

**Inboxes filled: none.** Every cross-project connection surfaced this sweep (`far-cry-2-vr` ↔
`XIII2003-vr` on OpenXR; the shader-compile-time table's Alan Wake corroboration) had already been
cross-linked directly between those projects' own repos by the concurrent `/gr` sweep before this
pass started — nothing was left to hand off.

**One thing worth naming:** this is the third day running that a `/gr` estate sweep and this `/sr`
sweep have landed on the same afternoon, and the second time a finding on one project was
independently verified or corroborated on a sibling within the hour (Alan Wake's NVAPI-table race on
2026-09-02's first entry; the OpenXR pose verification here). The account now has two research lanes
producing genuinely overlapping, mutually-checking output on the same day, which is a different
regime from the "one sweep, quiet day" shape most of this log describes.
