# id Tech 5

*One page per engine family this account has at least one conversion project on. This page holds
the **shared, cross-game truth** for the family; everything game-specific lives in each project's
`ENGINE-DOSSIER.md`, linked below. The [engines index](../engines-index.md) has the one-line
orientation row. Curated by the cross-project research sweep.*

## Identity

- **Engine:** id Software's id Tech 5 — here in Tango Gameworks' "STEM" branch used by The Evil
  Within. Source not released (unlike id Tech 1–4).
- **Render API:** Direct3D 11, 64-bit.
- **Known public VR path:** none turnkey. A strong candidate for a new adapter: typically Z-up
  basis, per-draw MVP delivery — see the dossier for the measured specifics.

## Our projects on this engine

| Game | Engine dossier | Project repo |
| --- | --- | --- |
| The Evil Within (2014) | [`ENGINE-DOSSIER.md`](https://github.com/TefMeister/the-evil-within-vr/blob/main/engine-research/ENGINE-DOSSIER.md) | [`the-evil-within-vr`](https://github.com/TefMeister/the-evil-within-vr) |

## Shared findings

*Seeded 2026-08-26; first populated 2026-09-01 from The Evil Within's dossier, so `n=1` by
construction.*

- **Per-draw MVP delivery is confirmed, and the shader-coverage gap is bounded rather than
  mysterious.** `[inferred-static 2026-09-01]` Of 168 vertex shaders: **112 place the MVP
  contiguously** and are handled; **22 carry no MVP at all** (a Domain-Shader group); and **34 carry
  an MVP with scattered rows**. Those 34 collapse into just **ten distinct `(cb0 size, mvp offset)`
  shapes**, one of which accounts for fifteen of them — so handling shapes in frequency order lifts
  shader coverage from 66.7% to 86.9%. The useful shape of this finding is that an "unexplained
  remainder" turned out to be a short, sortable list.
- **Record the limit alongside the number.** Those are *shader* counts, not the *draw* counts the
  coverage figure is measured in, and an offset table storing only a base offset cannot say where
  the remaining matrix rows sit — that needs the bytecode. Two different denominators for one metric
  is exactly the thing that reads as a contradiction six weeks later if it is not written down when
  it is still obvious.
- **Reflection and offset tables live on disk and can be read without launching.** See
  [read the shipped files before you attach anything](../techniques/#read-the-shipped-files-before-you-attach-anything)
  — this family has now contributed to that pattern alongside its successor, id Tech 6.

### The renderer is deferred, and that changes where the camera can be owned

`[verified-live 2026-08-21, from The Evil Within]` This branch of the family records its world on
**worker-thread deferred contexts** — hundreds of `ExecuteCommandList` replays per frame on the
immediate context, which also draws a minority directly (HUD and post). The world's per-object MVP
lives in a small **pool of constant buffers, one ring per worker thread**, created
`D3D11_USAGE_DEFAULT` with no CPU access and written through **`UpdateSubresource`** — so no
`Map`/`Unmap` hook can ever see the write, and its absence is an API rule rather than a missed hook.
The ownership proof that worked shadows those writes and patches **at record time**, once per eye,
into the mod's own per-thread scratch buffer. Full generalisation:
[deferred-context renderers](../techniques/#deferred-context-renderers-finding-the-world-and-patching-it-once-per-eye)
and
[identify a resource by how it is used](../techniques/#identify-a-resource-by-how-it-is-used-not-by-its-creation-descriptor).

**The per-eye maths falls out cheaply from per-draw MVP delivery.** Because every draw ends in
`M = P · V · M_model`, one constant per eye — `K_eye = P_eye · T_eye(±IPD/2 in x) · P⁻¹` —
left-multiplied onto every per-draw MVP produces correct stereo with **no per-object knowledge at
all**. That is the payoff of this delivery style, and it is worth confirming a family uses it before
choosing an approach. **Skinning happens in model space, before the MVP multiply**, in a separate
joint-palette buffer, so a camera-side change leaves animation untouched.

### The console and cvar culture is intact, and it is a real asset

`[verified-live]` The id console survives into this branch: `+com_allowconsole 1` at launch, then
`listcmds`, `devmapjump <stage>` to reach a scene in seconds, `noclip`, `g_fov`, third-person camera
cvars, HUD and shadow toggles, and the developers' own frame-capture-to-disk commands. A
deterministic unattended launch straight into gameplay is therefore available on this family without
writing an input harness first — with one caveat measured on The Evil Within: a **photosensitivity
warning splash was not dismissible by synthetic keyboard input, only by a synthetic mouse click**,
and the skip-intro cvars did not cover it.

**No stereo heritage here, unlike id Tech 6.** A full string scan of that title found zero
`stereo3d` / `oculus` / `openvr` / `vive` / `steamvr` / `stereoscopic` matches, so the
[dormant-stereo-path shortcut](../case-studies/id-tech-6-dormant-stereo.md) its successor offers
should not be assumed for this generation.

## See also

- [engines index](../engines-index.md) — the "id Tech 5" row.
- [id Tech 6](./id-tech-6.md) — the next generation, with its own page and case study; long-lived
  id Tech families inherit renderer code silently across generations.
