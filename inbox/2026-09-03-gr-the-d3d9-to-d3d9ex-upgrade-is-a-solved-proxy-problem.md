# The D3D9 → D3D9Ex upgrade is a solved proxy problem — every D3D9 project in the estate hits this wall

**From:** `/gr` (2026-09-03, estate sweep)
**Suggested home:** a method page, or the D3D9 section of whichever page covers getting a rendered
frame to a modern VR compositor. It is not engine-specific — it is a property of the D3D9 API and of
any `d3d9.dll` proxy.

## The wall

Every D3D9 game that eventually wants to hand a texture to an OpenVR/OpenXR compositor runs into the
same chain:

1. The documented no-copy route to the compositor is a **shared texture**: create it on the D3D11
   side with `D3D11_RESOURCE_MISC_SHARED`, take the `HANDLE` from
   `IDXGIResource::GetSharedHandle`, and open it from D3D9 via
   `IDirect3DDevice9Ex::CreateTexture(..., pSharedHandle)`.
2. **That requires a D3D9Ex device**, and games of this era create plain `Direct3DCreate9` devices.
3. The proxy can fix that for free — it already owns `d3d9.dll`, `IDirect3DDevice9Ex` **derives
   from** `IDirect3DDevice9`, so the proxy can call `Direct3DCreate9Ex` itself and hand the game the
   Ex object through the base vtable. The game is compiled against the base interface and need not
   know.
4. **Then `D3DPOOL_MANAGED` bites.** Microsoft's `D3DPOOL` reference, differences box:
   *"D3DPOOL_MANAGED is valid with IDirect3DDevice9; however, it is not valid with
   IDirect3DDevice9Ex."* Any `CreateTexture` / `CreateVertexBuffer` / `CreateIndexBuffer` asking for
   it fails.

Step 4 is where at least one project in the estate recorded the question as *"the difference between
a one-line proxy change and a resource-remapping project"* and made an instrumented launch a
prerequisite. **It should not be a prerequisite**, and that is the point of this note.

## Why the rewrite is safe, not a hack

Two quotes from Microsoft's own pages, which together explain the removal:

- `D3DPOOL`: applications should use MANAGED for most static resources *"because this saves the
  application from having to deal with lost devices. (Managed resources are restored by the
  runtime.)"*
- "Lost Devices (Direct3D 9)": *"A Direct3D 9Ex device never returns D3DERR_DEVICELOST."*

**MANAGED is a device-loss recovery mechanism, and a 9Ex device has no device loss.** The pool was
withdrawn because on 9Ex it has nothing left to do — so translating it away is the migration the
design implies, not a semantic compromise. `[reported 2026-09-03]`

## The one real difference, and the exact rewrite

Not all of MANAGED's behaviour is device-loss. From the same page: DEFAULT textures *"cannot be
locked unless they are dynamic textures"*, while managed resources can always be locked. So a naive
`MANAGED → DEFAULT` keeps every allocation succeeding and then fails at the first `Lock()`.

**The established rewrite is therefore `D3DPOOL_MANAGED → D3DPOOL_DEFAULT + D3DUSAGE_DYNAMIC`.** The
`DYNAMIC` usage is what restores lockability, and the page's pool × usage table confirms it is the
legal pairing (DEFAULT × DYNAMIC = yes; MANAGED × DYNAMIC is not permitted, so the rewrite can never
collide with a flag the game already set).

## Prior art, with a success rate and a failure list

`elishacloud/dxwrapper` ships exactly this as a `D3d9to9Ex` option, and its maintainer states the
strategy plainly: **override MANAGED to DEFAULT + DYNAMIC, following Special K**, which *"opened up
a lot more games"* — **7 of 8 tested worked**. `[reported 2026-09-03]`

The same source names what breaks, and this is the more useful half, because it replaces one vague
unknown with four observable ones:

- **paletted textures** are unsupported on 9Ex;
- **16-bit textures** only work in system memory;
- **D3DX functions** remain problematic (relevant to any 2008–2011 title with a D3DX dependency
  chain);
- **some titles fail outright at device-creation time.**

## Two more D3D9-side traps worth recording on the same page

Both already established elsewhere in the estate, and they belong beside this:

- **There is no `IDirect3D9KeyedMutex`.** `D3D11_RESOURCE_MISC_SHARED_KEYEDMUTEX` has no D3D9
  equivalent, so the synchronisation primitive every tutorial reaches for is unavailable to a D3D9Ex
  producer. The substitute is an `IDirect3DQuery9` event query plus double or triple buffering.
- **OpenVR issue #1253** (open): SteamVR keeps only the pose from the *last* `Submit`, so per-eye
  `Submit_TextureWithPose` ghosts. Submit both eyes together rather than racing per-eye pose timing.

## Why it is worth centralising

The estate currently has several D3D9 titles at different stages, and this chain is identical for
all of them — the API does not care which engine is above it. Recording it once means the next
project to reach step 4 reads the answer instead of scheduling a launch to discover the question.

There is also a scheduling consequence, in the estate's own gate language: **this moves a `[FLAT]`
item to "measurement, not gate".** The instrumented `D3DPOOL_MANAGED` count is still worth having —
it sizes how much `Lock()` traffic gets re-pointed — but it stops being something the Ex upgrade
must wait for, which removes a prerequisite from in front of static work.

## Sources

- [D3DPOOL enumeration](https://learn.microsoft.com/en-us/windows/win32/direct3d9/d3dpool) —
  Microsoft: the 9-vs-9Ex differences box, DEFAULT lockability, the pool × usage tables, and the
  device-loss rationale for MANAGED.
- [Lost Devices (Direct3D 9)](https://learn.microsoft.com/en-us/windows/win32/direct3d9/lost-devices)
  — Microsoft: "A Direct3D 9Ex device never returns D3DERR_DEVICELOST."
- [dxwrapper discussion #105](https://github.com/elishacloud/dxwrapper/discussions/105) —
  elishacloud: the `D3d9to9Ex` conversion, the MANAGED → DEFAULT + DYNAMIC strategy credited to
  Special K, the 7-of-8 result and the four named limits.
- [elishacloud/DirectX-Wrappers](https://github.com/elishacloud/DirectX-Wrappers) — the D3D9
  interface wrapper headers it builds on.
- The estate's own framing of the problem:
  [`enslaved-vr/engine-research/ENGINE-DOSSIER.md` §9](https://github.com/TefMeister/enslaved-vr/blob/main/engine-research/ENGINE-DOSSIER.md),
  and the full write-up in
  [that project's 2026-09-03 topic](https://github.com/TefMeister/enslaved-vr/blob/main/external-research/topics/2026-09-03-the-d3d9ex-upgrade-is-a-solved-bounded-proxy-problem.md).
