# RE Engine: bullet spread is a scattered ROTATION the bullet is BUILT with — and how to cancel it (2026-09-21)

From: modding lane (`/pd` + live tests by Tefa), `re-village-scope-vr`. Dossier §9bd–§9bl there.
For: the library — an RE Engine family finding **and** an engine-agnostic method.

## The finding (RE Village, `re8.exe`, REFramework pd-upscaler fork)

`app.WeaponGunCore` fires a shot in this order `[verified-live 2026-09-20, n=2 traces]`:

```
expendBullet -> createBullet(via.Ray, System.Boolean)
             -> createBulletImple(via.vec3, via.Quaternion, via.GameObject, System.Boolean)
             -> setupDiffusion(via.vec3, via.Quaternion, via.Quaternion)      (shootCommon runs after)
```

- The **ray** handed to `createBullet` is the exact muzzle axis — 0.000° off on 8 of 8 shots, including
  shots with 14.6° of scatter `[verified-live 2026-09-21, n=8]`. Layout: origin is a **padded** vec3,
  the direction is at **+16**.
- Inside `createBullet` the game turns that ray into a rotation and **scatters it**. Call the result A.
- **`createBulletImple` builds the bullet with A** — A is bit-identical in both calls.
- `setupDiffusion` is then handed A again plus **B, the clean rotation**. B is exactly the
  shortest-arc rotation from `+Z` onto the ray: `normalize(-d.y, d.x, 0, 1 + d.z)`, matching to four
  decimals on every shot `[verified-numerically 2026-09-21, n=3; live formula check n=10, no warnings]`.
- Measured scatter for the sniper rifle: **aimed 0.005° avg; hip 8.4° avg, 14.9° worst**
  `[verified-live 2026-09-20, n=10]`. Aiming does not flip `isRestrictAimShake` (always true) or
  `isReduceRecoil` (always false); `enableRestrictAimShake()` / `enableReduceRecoil()` take no
  arguments — they are queries, not setters. `GunSpec.get_diffusionRadius` / `get_isDiffusion` are
  **never called at firing time** `[verified-live 2026-09-20, n=2]`.

**The cancel that works** `[verified-live 2026-09-21, n=5 shots + the wearer's eyes]`: in a native
pre-hook on `createBulletImple`, overwrite the rotation argument with the clean rotation computed from
the ray captured in the `createBullet` pre-hook one call earlier. Five hip shots carrying
11.7 / 0.5 / 11.2 / 5.4 / 5.6° of scatter were each built 0.000–0.040° off, and flew straight.

**What does NOT work, each tested:** writing at `setupDiffusion` in either direction (lands, log reads
`11.188 -> 0.000`, bullet unaffected — it already exists); skipping `setupDiffusion`; overriding the
spec getters; and **anything from Lua** — a value-type argument cannot be written from a REFramework
Lua hook by either route (`args[n] =` reassignment, or `sdk.to_valuetype(...):write_float`, which
returns a copy) `[verified-live 2026-09-20]`. Reading them from Lua works fine (`valuetype` route).

⚠️ Native-hook trap: in the plugin API's pre-hook, `arg_tys[i]` are **handles, not pointers** (API.h
says so); casting one to `TypeDefinition*` crashed the game on the first call. `argc` does **not**
equal 2 + declared parameters (it was 7 for a 3-parameter method) — validate argument *data*, e.g.
a rotation must be a unit quaternion within 0.001.

Likely shared across the RE Engine family (RE2/RE3/RE7/RE4 all use `WeaponGunCore`-style gun cores);
⚠️ **unchecked outside RE8**. RE2's type dump shows `set_Diffusion` / `get_Diffusion` and no trace of
its firing path has been taken.

## The method, which is engine-agnostic

1. **Measure before hunting.** Find any function handed both an "intended" and an "actual" direction
   and log the angle per shot. Here that gave aimed-vs-hip numbers within one test, and the measure
   stayed right all evening while five theories about the mechanism were wrong.
2. **Trace the ORDER, then say which step BUILDS the projectile.** A step that *carries* the scatter
   measures beautifully and changing it does nothing if the projectile already exists. Four builds
   died of this with the correct order already written down.
3. **Identify the scattered value by arithmetic on logged data**, not by name or position: compute the
   clean value yourself and see which logged candidate matches it on accurate shots and diverges by the
   measured scatter on inaccurate ones.
4. **Every lever re-measures after it writes** and logs both numbers, so "no effect", "wrong choice" and
   "worked" are distinguishable without asking a person to judge bullet holes.
5. **Ask for several inaccurate shots, not one** — scatter is random per shot and a single hip shot can
   be a tight one (it was, twice).

Now in the lanes plugin as `docs/PROTOCOL.md` §11.
