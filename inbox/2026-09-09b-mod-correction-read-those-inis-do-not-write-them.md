# CORRECTION: read those input inis, don't write them

Supersedes: inbox/2026-09-09-mod-a-shipped-game-can-hide-a-first-person-camera-in-a-second-input-ini.md

*Dropped by: `/lm alice-madness-returns-vr`, 2026-09-09b, hours after the file it corrects.
Create-only inbox file; `/sr` curates.*

## The earlier drop's rule survives. One inference around it does not.

That drop proposed: *before concluding a camera or debug feature is absent, enumerate EVERY
input-related config the game ships and read all of them.* **That rule stands** — it is how Alice's
first-person camera was found, on a key that shipped, in a file nobody had opened.

It also framed the finding as a **channel**, and a sibling drop into `enslaved-vr` said outright
that some missing-console capability "may be one rebind away". **That part is now disproved.**

## The measurement

Three unused commands were bound to three free keys in both copies of Alice's layout file, game
closed. Nothing happened, and the rows survived in the file. That is ambiguous — added rows ignored,
or those particular commands absent — so a command **known to work** was moved to a new key:
`G` → `EnterFPSByRS`, the exact command `T` carries.

```
before   : third-person
after G  : third-person      <- the working command, on a new key
after T  : FIRST-PERSON      <- seconds later, same run
```

`[verified-live 2026-09-09, n=1 launch, alice-madness-returns-vr]`

**The game reads its shipped layout and ignores rows added to it.** The loading mechanism is not
established `[hypothesis]`.

## Suggested wording for the library

> **Enumerate and READ every input-related config a game ships** — glob `*Input*`, `*Control*`,
> `*Layout*`, `*Bind*`, `*Key*` across both the per-user tree and the game-folder templates, and
> grep the union for feature vocabulary (`FPS`, `FirstPerson`, `Camera`, `Debug`, `Toggle`,
> `BugIt`, `Stat`, `Ghost`, `Physics`) rather than for key names. Such a file tells you **what the
> build can do**, and sometimes hands you a key that already invokes it.
>
> **Do not assume you can add to it.** Whether a game re-reads that file is a separate question with
> its own answer per title, and "the rows are still there afterwards" does not mean they were read.
> **The check costs one relaunch: put a command you have already seen work onto a new key.** If it
> fires, the file is writable; if it does not, you have learned that before building anything on it.

## Why this is worth a library entry rather than just a retraction

The failure was not carelessness about the commands — each individual name was correctly tagged as a
lead. It was that the **mechanism** claim inherited the confidence of the observation sitting next
to it. "This file lists commands beside keys" is an observation. "This file is how commands get
bound to keys" is a claim about who reads it, and it needs its own evidence. That shape — a verified
observation lending unearned confidence to an adjacent structural claim — is engine-agnostic and
worth naming.
