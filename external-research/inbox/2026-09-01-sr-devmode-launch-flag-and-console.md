# Dunia ships a developer mode behind a `-DEVMODE` launch flag, and a console with self-listing commands

**From:** `/sr` cross-project sweep, 2026-09-01 (dev PC)
**For:** `/gr far-cry-2-vr` — research it properly and curate it into `external-research/`.
**Confidence:** `[reported]` — community guides and forum threads, mostly 2008–2022, none tested by
us and none checked against the build installed here.

## What the sweep found

- **`-DEVMODE` as a command-line parameter** is widely described as unlocking Far Cry 2's developer
  mode (and all levels), after which various keys trigger cheats.
- **The console opens with `~`**, and — the useful part — **`?` lists available commands** while
  **TAB completes / enumerates** them. A console that enumerates itself is worth more than any
  second-hand command list.
- Several cheat behaviours are also reachable as command-line parameters rather than console
  commands (god mode, unlimited ammo, weapon reliability, AI ignoring the player, all weapons).
- **One warning repeated across sources:** do not enable god mode or unlimited ammo before finishing
  the tutorial, or the game gets stuck. Worth heeding on a save you care about.

## Why this matters to this project right now

This project just built **head tracking and verified it numerically** (2026-09-01), with no headset
on the dev machine and no live run. The next step is inevitably a live one, and a developer console
changes what that session can accomplish:

- **`?` / TAB self-enumeration means a full command and cvar capture is a five-minute job.** That is
  the single highest-value first action, and it is the thing that later makes questions answerable
  without another launch. Capture it to a file if the console supports it.
- **Look specifically for a camera-position printer.** On other projects in this estate a console
  that prints the live camera became a permanent ground-truth instrument — and it is what makes a
  head-tracking composition checkable arithmetically rather than by feel. Given this project's
  numerical harness already solves the camera position out of the matrix, a console readout gives
  an **independent second opinion** on that solve, which is exactly the check the two composition
  bugs found this week would have wanted.
- **A frozen-time or free-camera command, if one exists, is worth more than any cheat.** Check for
  one while enumerating.

## What to be careful about

1. **`-DEVMODE` sources are old.** Confirm on the currently installed Steam build before relying on
   it, and note that this game has a known launcher/DRM history worth not disturbing.
2. **Enumerate, do not trust lists.** Community command lists for this game are recycled between
   sites and are not obviously first-hand. The console's own `?` output is the authority.
3. **Change one thing per launch.** If `-DEVMODE` is combined with other flags, a positive cannot be
   attributed.

Sources: [AlteredGamer — devmode and cheats](https://www.alteredgamer.com/far-cry-2/22835-cheats-devmode-cheat-codes-and-shortcuts/) ·
[AnandTech forums — Far Cry 2 console commands](https://forums.anandtech.com/threads/far-cry-2-console-commands.230928/) ·
[GameRevolution — Far Cry 2 PC cheats](https://www.gamerevolution.com/guides/42156-far-cry-2-pc-cheats)
