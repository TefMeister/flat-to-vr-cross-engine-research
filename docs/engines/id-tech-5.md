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

`[verified-live 2026-08-21, n=1 game]` — **The Evil Within only**, which is the whole family here, so
read this as one game confirmed many times rather than a pattern seen across siblings. The id console
survives into this branch: `+com_allowconsole 1` at launch, then `listcmds`, `devmapjump <stage>` to
reach a scene in seconds, `noclip`, `g_fov`, third-person camera cvars, HUD and shadow toggles, and
the developers' own frame-capture-to-disk commands. It has carried an unattended launch harness since
2026-08-21, so the confirmation is repeated rather than a single lucky run. A deterministic unattended
launch straight into gameplay is therefore available on this title without writing an input harness
first — and `[hypothesis]` on any other id Tech 5 game, since no second one has been tried — with one caveat measured on the same game: a **photosensitivity
warning splash was not dismissible by synthetic keyboard input, only by a synthetic mouse click**,
and the skip-intro cvars did not cover it.

**No stereo heritage here, unlike id Tech 6.** A full string scan of that title found zero
`stereo3d` / `oculus` / `openvr` / `vive` / `steamvr` / `stereoscopic` matches, so the
[dormant-stereo-path shortcut](../case-studies/id-tech-6-dormant-stereo.md) its successor offers
should not be assumed for this generation.

### The shaders were on disk all along — `.tangoresource` decoded, 2026-09-03

`[verified-numerically 2026-09-03]` The Evil Within ships its shader bytecode: `base/common.tangoresource`
(magic `0x2394ABCD`, big-endian entry count, paired name records with a trailing hash, entries stored
as **headerless raw deflate**, and a big-endian offset/csize/usize/id table at the end) yields
**2,785 DXBC shaders with `RDEF` intact**. The project's own `constantBufferV` appears in 1,208 of
them at `cb0` with its rows *named* (`mvpmatrixx/y/z/w`) at explicit byte offsets, and the runtime
table it had built live matches **167 of 168 by hash with zero disagreements** — two unrelated
methods, one table. Consequence: "scattered" MVP rows are almost always one thing, **z and w
swapped** (33 of 34), the patch now reads all four offsets instead of refusing non-contiguous
layouts (shader coverage 66.7 % → 86.9 %), and a latent out-of-bounds write in the old bounds check
— valid only while offsets ascend — was found by inspection. 249 of 9,001 entries (2.8 %) do not
inflate as raw deflate and are unexplained; no shader was among them. Method note worth more than the
finding: the first scan looked for zlib framing, which headerless deflate does not have, and would
have been written up as "not zlib" — a test that could not have produced a positive.

### 2026-09-04: a patch-coverage residual that was a second render path, not a ceiling

`[measured 2026-09-03, n=167 shaders]` · `[verified-live 2026-09-03, n=3 scenes]` The Evil Within's
per-draw matrix patcher leaves a fraction of draws unpatched, and that fraction was carried for a while
as a possible hard limit on the technique — the sort of number that quietly decides whether a project
is viable. It is not a limit. The pool the patcher matches against registers only the **large shared
`DEFAULT` world buffer**, so a miss on a small per-shader `DYNAMIC` `cb0` is the expected outcome, not a
failure: declared `cb0` sizes across all 167 known shaders run 0–352 bytes, none of them inside the
pool's size window. The residual's share also **swings with scene and camera** (13% → 19%), which is
itself the signature of a scene property rather than a fixed ceiling.

**The transferable half is how long the question stayed open.** The proxy had bucketed the missed draws
by size and usage flags for several rounds — everything needed to answer *do the missed draws carry
world geometry?* — but printed the table **only while nothing had been patched**. The moment patching
started working, the diagnostic disappeared. See
[the diagnostic that is gated on the failure it was written to explain](../techniques/README.md#the-diagnostic-that-is-gated-on-the-failure-it-was-written-to-explain).
It now prints periodically and at shutdown, with per-bucket draw sizes and vertex-shader hashes — and
**it has now run** `[verified-live 2026-09-04, n=1 launch]`.

**⚠️ The answer sharpens the paragraph above rather than confirming it: the residual is not a ceiling,
but it is not harmless either — the missed draws carry real world geometry.** Against roughly 590,000
patched draws per five seconds, the skipped ones are a single category, and several of the per-shader
`DYNAMIC` `cb0` buckets carry substantial indexed and non-indexed geometry with vertex counts up to six
figures. These are matrix-bearing draws whose constant buffer the shared-pool patch never intercepts —
not draws without a transform.

**The confirmation was visual and took one launch.** With a deliberate 90° yaw applied to every draw the
patch reached, the game's opening scene rendered radically transformed **with unrotated fragments
through it** and an upright, detached character head — the uncovered geometry, rendering in place. That
technique generalises and is now on the techniques page as
[the cheapest coverage test is an absurd transform](../techniques/README.md#-the-cheapest-coverage-test-is-an-absurd-transform-and-it-is-a-picture).

**⇒ For this family, a stereo build must extend coverage to the per-shader `DYNAMIC` `cb0` path, not
only the shared `DEFAULT` pool.**

**And that path is now built** `[measured 2026-09-04, n=167 shaders]` · `[compile-verified
2026-09-04]`, not run. Of the matrix-bearing shaders in the live table, **42% declare their constant
buffer at one of the sizes the coverage test showed carrying geometry**, and every one of them already
had a complete reflected layout on record — so they had been patchable all along and only the *buffer*
was out of reach. The gap was never about shaders: a `DEFAULT` buffer is written through
`UpdateSubresource` and a `DYNAMIC` one through `Map`/`Unmap`, and the pool watched only the first.
The fix is one more shadow **source**, with the draw-time path unchanged. ⚠️ That 42% is a **shader**
figure and does not convert into the draw figures quoted above — different populations. General form:
[enumerate every CPU write path to a constant buffer](../techniques/README.md#enumerate-every-cpu-write-path-to-a-constant-buffer-before-believing-your-coverage).

Also on this project as of 2026-09-04: every proxy knob is read from an ini beside the executable
rather than from the environment, after three launches silently ran an experiment as an identity
transform because a **storefront-launched game does not inherit a user shell's environment**
([techniques](../techniques/README.md#configure-injected-code-from-a-file-it-reads-itself-not-from-environment-variables)).

### 2026-09-05: that second path failed on its first run, and the reason is a rule about deferred contexts

`[verified-numerically 2026-09-05]` The `Map`/`Unmap` path above ran live on 2026-09-04 and produced
**zero** shadow writes against **2,787,733** pairing-table overflows — a total failure, which is at
least an honest one. It matters to this whole engine family because the cause is a direct consequence
of the deferred-context architecture described at the top of this page.

**The pairing between `Map` and `Unmap` cannot be keyed on the buffer.** `WRITE_DISCARD` renames per
context, so several deferred contexts may legally hold an outstanding map of the *same* buffer at
once — with around six workers plus the immediate context and a few dozen registered buffers, that is
the normal state, not an edge case. A table that claims a slot by resource pointer and searches for it
at `Unmap` matches the wrong entry and saturates. **The identity that is unique by construction is the
pair `(context, resource)`**, since the API allows one context only one outstanding map of a
subresource. Full write-up, including the lock-free claim/release ordering and a pointer-width trap
caught before it shipped:
[an in-flight map's identity is `(context, resource)`](../techniques/README.md#-and-an-in-flight-maps-identity-is-context-resource--never-the-resource-alone).

**Two engine-level cautions that come with it, both still open:**

- The shadow copy behind the table is still **one window per buffer**, so two contexts writing the
  same buffer simultaneously cannot both be represented. That limitation is unchanged; its counter
  only starts meaning anything now that writes reach it at all.
- `Map`/`Unmap` are hooked **once, on the immediate context, and never late-hooked**, unlike the
  neighbouring `UpdateSubresource` path. Deferred contexts come from a different vtable, so *"the fix
  works"* and *"our hook never sees these calls"* are still indistinguishable until the new
  maps-seen / unmaps-seen counters separate them.

⚠️ **And the diagnosis this replaces was wrong in an instructive way.** The 2026-09-04 write-up blamed
a *"thread-ring pool exhausted (8 distinct threads seen)"* line printed in the same diagnostic block.
That line belongs to an unrelated subsystem — draw-time scratch buffers — and sizing that pool would
have changed nothing. `[disproved 2026-09-05]` General form:
[a log line that co-occurs with a failure is not an explanation of it](../techniques/README.md#a-log-line-that-co-occurs-with-a-failure-is-not-an-explanation-of-it).

## See also

- [engines index](../engines-index.md) — the "id Tech 5" row.
- [id Tech 6](./id-tech-6.md) — the next generation, with its own page and case study; long-lived
  id Tech families inherit renderer code silently across generations.
