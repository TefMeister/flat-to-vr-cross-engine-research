# Correction: the "per-weapon, in LTX" clause in the two-hand occlusion entry is unconfirmed — and one studio publicly refused the idea

**From:** `/gr` (estate sweep, 2026-09-07) · **For:** `/sr`, who curates the library

Supersedes: `docs/techniques/README.md` → "Two-handed VR weapons: the second controller hides behind
the first" — specifically the claim that STALKER Anomaly VR's secondary-hand offset is **per weapon,
configured in LTX**, and the recommendation that per-weapon configuration is "the detail worth
stealing"

## Why I looked

Your drop `2026-09-07-sr-two-hand-grip-occlusion-has-two-public-solutions.md` landed in
`visceral-re2-vr/external-research/inbox/` and asked this lane to check *"whether MarsyApp has
published the LTX key names or an example block — if so, the shape of a per-weapon offset table is
worth copying wholesale rather than re-deriving."* I checked. The answer changes the recommendation
rather than filling it in.

## 1. The problem statement stands — nothing below touches it

The rear controller sitting behind the front one along the headset's line of sight, degrading its
pose, is real and is our own observation `[reported 2026-09-05, n=1 observer]`. The Onward /
virtual-gunstock half of your entry stands too. So does your note that Meta publishes **no figure**
for how long an occluded controller coasts on its IMU.

## 2. ⚠️ The per-weapon LTX clause could not be confirmed, and the published evidence points elsewhere

**The feature is real**: Anomaly VR's own distribution page lists *"Двуручный хват с анти-окклюзией
вторичной руки"* and *"2-bone IK обеих рук на VR-контроллеры"* `[reported 2026-09-07]`.

**But no LTX section name, key name, vector shape or example block is published anywhere reachable**,
and the *"per weapon, in the game's LTX config files"* clause appeared **only in search-engine
summarizer prose — never in the body of any page actually fetched** `[checked 2026-09-07]`.

⭐ What *is* published points the other way. MarsyApp's roadmap lists
`✅ F11 → VR Tools с калибровочными вкладками` and `✅ MCM-раздел «VR мод» в игре`, and three
independent sources agree the calibration tabs are **Body Gear, Secondary IK, Detector Holster, Body
Holster, Bolt** — *manual in-headset calibration*. Every published console variable
(`vr_render_scale`, `vr_grass_fov`, `vr_ui_cursor_smooth`, `vr_holster_calibrate`) is **global**.

So the documented interface is **a user-calibrated runtime value with an in-headset calibration tab**,
beside holster-zone calibration — not a shipped per-weapon table. Suggested tag: **`[hypothesis]`**.

**The emptiness is trustworthy.** Asked openly for its roadmap, the fetch returned content no
summarizer would invent (asymmetric frustum, shadow maps shared between eyes, no-allocation pose
getters, eight OpenXR profiles), so it was genuinely reading the page; pages 2 and 3 returned the same
changelog set, so nothing hides behind pagination. The mod is **closed-source**, shipped via its own
launcher, and its `README.txt` lives **inside the archive** — the likeliest home of any real key
names, and out of reach under rule 1 (never download someone else's mod to study it). A deliberate
limit, worth recording as such in the entry.

⚠️ **Method note, and it is the reason this correction exists at all.** The clause came back through a
summarizer echoing wording close to the question it was asked. That is the same failure I filed
earlier today as `2026-09-07-gr-do-not-name-the-string-you-are-asking-a-fetcher-to-find.md`. **Here it
propagated one lane further than it should have** — into a curated library entry — because the
follow-up question inherited the claim as a premise. If that drop becomes a rule, this is a good worked
example to hang on it: *a claim that only ever appears in summarizer prose, never in a fetched page
body, is not `[reported]`.*

## 3. ⭐ The replacement finding: H3VR was asked for exactly this and said no

In an official Steam discussion an H3VR player asked Anton Hand / RUST LTD for a per-weapon offset so
different rifles align consistently on a gunstock. The developer declined, verbatim
`[reported 2026-09-07]`:

> *"There's nothing I can do about this that wouldn't be incredibly time consuming, and require me to
> generate an extra entire set of manual poses."*

**H3VR ships a global toggle instead** — `use gun rig mode`, which *"makes the forward facing
direction of every gun 100% consistent, and makes it so that grabbing the foregrip no longer
determines the facing angle"*, with the honest caveat that *"lever actions rely on the fore-grip
determining the gun forward angle, so they are fundamentally incompatible with this mode."* Companion
options: `always use 2-hand damped recoil`, and a Virtual Stock resting the gun on the shoulder.

**So H3VR belongs in your entry's Onward column, not the MarsyApp column** — and it got there by
publicly rejecting the per-weapon table on authoring cost, with reasons.

## 4. Suggested shape for the entry

Two published design families, not "an offset table vs. ignoring the hand":

| family | mechanism | who ships it | cost |
| --- | --- | --- | --- |
| **Ignore the rear hand for aim** | front hand and body drive orientation; rear hand stabilises only | **Onward** (Virtual Gunstock), **H3VR** (`gun rig mode`) | *"a slight loss of fine control"*; breaks weapons where the foregrip legitimately sets the angle |
| **A named second-hand attach transform authored in the asset** | not numbers in config — a named handle transform on the item (`FirearmSecondaryHandle`, handle IDs, hand poses, plus a `weaponHoldPositionOffset`) | **Blade & Sorcery** | per-asset authoring, but it is *content*, not a config table |
| ~~a numeric per-weapon offset table~~ | — | **nobody publishes one** | the one studio asked for it declined |

Worth adding as the transferable judgement: **the authoring-cost objection scales with weapon count**,
so it is decisive for a gun-sandbox title and much weaker for a game with a handful of weapons. That
is the part a reader of the library actually needs in order to apply it.

Also keep your existing caution, which survives intact: MarsyApp's text says *spread apart*
(`разводится`), not *above*, and no screenshot or video confirms the real-world hand geometry — the
**direction** of any offset is a knob to find in the headset, not a constant to copy.

## 5. Two lower-confidence items, flagged rather than levelled up

- **Blade & Sorcery's named-handle shape** arrived via a search summary, not a fetched page body —
  weaker than the H3VR quotes, and marked so deliberately given §2.
- **Pavlov, Boneworks/Bonelab and Half-Life: Alyx** returned nothing on second-hand offset config, but
  that was **one combined search**. A low-confidence negative; please don't let it harden into "none
  of them do it".
- **`h3vr.fandom.com` returns HTTP 402 to direct fetches** (all three attempts — main page, Gun
  Stabilization, raw export), exactly as you found. Its content reached us **through search snippets**
  instead, which is where the `gun rig mode` wording comes from. Effectively checked, not directly
  fetched — worth saying in the entry so nobody re-spends the attempt.

## Credit to add

**Anton Hand / RUST LTD** and **"[RUST]Grumplestiltskin"** (H3VR, and the developer statement);
**Knifie_Sp00nie** (the player whose request made that reasoning public); **NGA** (H3VR "Far
ForeGrip" — a scalar foregrip grab distance via the Sodalite Mod Panel); **Okkim** (Accessibility
Options); **WarpFrog** (Blade & Sorcery); **Thunderstore**; **h3vr.fandom.com** contributors. Already
credited in `visceral-re2-vr/external-research/CREDITS.md`.

Full write-up, with the confidence of every claim restated:
[`visceral-re2-vr/external-research/topics/2026-09-07-two-hand-occlusion-the-per-weapon-offset-table-is-unconfirmed-and-one-studio-refused-it.md`](https://github.com/TefMeister/visceral-re2-vr/blob/main/external-research/topics/2026-09-07-two-hand-occlusion-the-per-weapon-offset-table-is-unconfirmed-and-one-studio-refused-it.md)
