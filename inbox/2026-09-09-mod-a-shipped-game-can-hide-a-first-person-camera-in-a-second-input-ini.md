# A shipped game can hide a first-person camera in a SECOND input ini, on a controller chord

*Dropped by: `/lm alice-madness-returns-vr`, 2026-09-09. Create-only inbox file; `/sr` curates.*

## The finding

**Alice: Madness Returns has a working first-person camera. It needs no mod, no rebind and no
console. It is on the `T` key, in the retail build, today.** `[verified-live 2026-09-09, n=1 launch]`

The project had spent four sessions on camera control and had concluded, correctly, that this build
exposes no developer console. It had read `AliceGame\Config\AliceInput.ini` several times. It had
**never opened `AliceControlLayout.ini`** — a second file, in the same directory, which is where
this game keeps its *action* bindings. `AliceInput.ini` holds only axes and aliases.

```
KeyBindArray1=(Name="T",Command="EnterFPSByRS | OnRelease ToggleCloseFollowCamera")
KeyBindArray1=(Name="XboxTypeS_RightThumbstick",Command="ToggleGhost | OnRelease ToggleCloseFollowCamera |EnterFPS")
```

The action's primary home is the **right-stick click** — a controller chord, exactly the case the
PLAYBOOK already warns that keyboard probing will never discover. What is new here is the second
half: it was *also* on a plain letter key the whole time, and the reason nobody pressed it is that
the file naming it was never opened.

## Why this is worth generalising

Three separate rules already in the library each nearly caught this and none of them did:

1. *"UE3 titles often reach debug features by a controller chord rather than a key."* True here, and
   it stopped at "so use a pad" rather than "so read where the chords are declared".
2. *"A binding surviving in a shipped ini is not evidence the feature is live."* Also true, and it
   is a rule about **not over-trusting** an ini — which quietly discourages reading more of them.
3. *"The console is absent in this build"* was established well, with five candidate causes
   excluded. For about a day it was read as *"the game's commands are unreachable"*, which does not
   follow: **a key binding that names an engine command is a command channel with no console in the
   path.**

## Suggested rule for the library

**Before concluding a camera or debug feature is absent from a game, enumerate EVERY input-related
config file the game ships and read all of them — not just the one named `<Game>Input.ini`.** Glob
for `*Input*`, `*Control*`, `*Layout*`, `*Bind*`, `*Key*` under both the live per-user config tree
and the game-folder template tree. Then grep the union for the feature vocabulary
(`FPS`, `FirstPerson`, `Camera`, `Debug`, `Toggle`, `BugIt`, `Stat`, `Ghost`, `Physics`) rather than
for a key name.

The cost is one `grep` over a handful of text files, before any launch. The thing it found here is
the single most useful capability discovered on this project.

## What is NOT claimed

- Only `EnterFPSByRS` was actually run. The same file names `ChangeCameraMode`,
  `ToggleCloseFollowCamera`, `TogglePOI`, `ToggleGhost`, `togglephysicsmode`,
  `BugItForGameController` and `StatUnitAndStatFPS`, and **all of those remain leads, not
  evidence** — rule 2 above still stands. `[reported 2026-09-09]`
- Whether other engines split their bindings the same way is **untested**. This is one UE3 title.
  UE3 is common enough across the estate (`enslaved-vr`, `alice-madness-returns-vr`) that it is
  worth one check per project; whether it generalises past UE3 is unknown. `[hypothesis]`
- Nothing here says the first-person camera is *good*. It aims finely and survives walking; comfort,
  head height and combat behaviour are all untested.
