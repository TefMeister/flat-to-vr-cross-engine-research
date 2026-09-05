# Two-handed VR weapons: stack the hands, do not line them up (engine-agnostic, from the modding lane)

Source: Tefa, in the headset, RE Village scope session 2026-09-05 ~23:05. `[reported]`

- **Observation:** with a two-handed weapon held naturally, the left (fore-grip) controller ends
  up directly behind the right one from the headset cameras' point of view. It is occluded and
  tracking degrades at once; the weapon jitters or swings. Seen live that night, cost a retake.
- **Requirement (universal, all our games):** derive the in-game two-hand pose from a real-world
  pose where the LEFT hand is held ABOVE the right, so neither controller hides the other, while
  the weapon still looks and aims correctly in the game.
- **Known implementation to study:** the STALKER Anomaly VR mod by MarsyApp is reported to do
  exactly this `[reported]`. Candidate for a `/gr` pass and a library page under grip/IK.
- Suggested home: the two-hand grip / weapon IK topic, cross-linked from each project's grip row.
