# The console unlock is `devmodeon` typed in-game (not the launch flag), it prints a *misleading error on success*, and TAB enumerates everything

**Status:** 🆕 new · **Priority:** high, and it is cheap. Roadmap item 3 (head tracking) is **built
and numerically verified but has never been run** — the next session on this project is a live one,
and a working console with self-enumeration is the difference between that session producing a
measurement and producing an impression.

## What was found

Far Cry 2's Dunia build ships a developer console that is reachable on retail, with no mod:

- **Open it with `~`.** ⚠️ **Layout-dependent** — a Spanish-keyboard user reports it opening on `¶`.
  See the trap section below; this is the third independent instance of it in this estate.
- **Unlock the hidden commands by typing `devmodeon` in the console.** This is the correction that
  matters: the sweep drop that prompted this topic recorded a **`-DEVMODE` launch parameter**, and
  that flag is separately documented as unlocking *all levels*. The thing that exposes the extra
  **console commands** is the in-console `devmodeon`. Treat them as two different switches until
  somebody has tested both.
- **🪤 `devmodeon` prints an error when it works.** The community instruction is explicit: *"It will
  say 'Unknown command: devmode'. Do not pay attention to this message."* A success that reports
  failure is exactly the shape that gets a working mechanism written off — see below.
- **`?` and `help` describe commands; TAB autocompletes and enumerates them.** A console that
  enumerates itself is worth more than any second-hand list, and it makes a complete capture a
  five-minute job.
- Confirmed command names in the wild are mostly renderer knobs — `gfx_Bloom`, `gfx_Hdr`,
  `gfx_MaxFPS`, `gfx_DisableShadowGeneration`, `gfx_WaterReflectionQuality`, `Gfx_Showconfig` — plus
  `Exec` (run a config file), `Quit` / `QuitToMainMenu`, and, post-`devmodeon`,
  `domino_SkipOpeningSequence`.

## Why this matters for this project specifically

**1. `Gfx_Showconfig` and the `gfx_*` family are directly on the roadmap.** This project's own
research already records that **shadow quality must stay Medium, not High** (the HelixMod topic), and
flags water reflections and fire/smoke as trouble passes to expect in a per-eye override. Those are
`gfx_DisableShadowGeneration` and `gfx_WaterReflectionQuality` — the exact knobs, settable live,
without a rebuild or a settings-menu round trip. That turns "expect trouble in these passes" into
"toggle each one and look".

**2. `Exec` is a scripting foothold.** A config file of `gfx_*` settings applied in one command makes
a stereo test repeatable across launches, which matters when the thing being judged is subtle
(ghosting, a pass that renders per-eye wrongly).

**3. The head-tracking build needs ground truth, and the console is the cheapest source.** The
composition currently solves the camera's world position out of `M` — and two bugs in that solve were
already caught by a numerical harness, one of them an inverse-composition error that *presents
exactly like a handedness problem*. **If the console can print a camera or player position, that is
an independent second opinion on the solve**, obtained without a headset. Hunting for such a command
should be the first thing done after `devmodeon` — enumerate with TAB and look for anything matching
`pos`, `cam`, `view`, `player`, `teleport`, `debug`.

**4. `gfx_MaxFPS` is a frame-pacing lever.** This project already has a topic on SteamVR frame timing
and the vsync gotcha — that the game's own D3D9 `Present` must not vsync to the desktop refresh, or
it fights the compositor's cadence. A live frame cap is a way to probe that without touching code.

## 🪤 Two traps, both of which this estate has now paid for elsewhere

**The console key is layout-dependent, and the physical scancode is the stable fact.** The `¶` report
here is the third independent instance: DOOM (2016) cost a session to it, and then cost a *second*
correction when the active keyboard layout turned out to differ between two launches of the same game
on the same machine, hours apart. The durable fix, already written up in the cross-engine library, is
to **send the physical scancode of the key left of `1` (`0x29`) and keep a virtual-key constant out
of the path entirely**. If this project ever automates the console, do that from the start rather
than resolving a VK.

**A success that reports an error is a negative-result trap.** `devmodeon` answering *"Unknown
command: devmode"* is precisely the situation the library's control rules exist for: **do not record
the unlock as failed on the strength of the message — verify against something you can see**, by
TAB-enumerating before and after and comparing what is offered. If the command list grows, it worked,
whatever it printed.

## What is NOT established

Everything here is `[reported]`, from community sources spanning roughly 2008–2022, **none of it
tested by us and none of it checked against the Steam build installed on this machine**. Specifically
unknown: whether `devmodeon` still works on the current build; whether a camera or position readout
exists at all (no source names one); whether `-DEVMODE` and `devmodeon` overlap; and whether the
`gfx_*` values survive a level load. One warning worth heeding from the sources — do not enable god
mode or unlimited ammo before finishing the tutorial, or the game can get stuck.

## Concrete next steps, cheapest first, for the next live session

1. Open the console, **TAB-enumerate and capture the list**, then type `devmodeon`, **enumerate
   again, and diff the two**. That single comparison establishes whether it works, immune to the
   misleading error message.
2. Search the enumerated list for a **position or camera readout**. If one exists, it is the
   independent check on the head-tracking position solve, and worth more to this project than
   anything else on this page.
3. Try `Gfx_Showconfig`, then `gfx_DisableShadowGeneration` and `gfx_WaterReflectionQuality` against
   the pass-trouble predictions already in this lane.
4. Record which key actually opened the console **and the layout that was active at the time** —
   both, because the DOOM case proved the layout alone can change between launches.

## Sources

- [Steam Community — Far Cry 2 Crew: "devmodeon" exposes new console commands](https://steamcommunity.com/groups/FarCry2Crew/discussions/6/2269193447675170650/)
- [AnandTech forums — Far Cry 2 console commands](https://forums.anandtech.com/threads/far-cry-2-console-commands.230928/)
  (the `gfx_*` list, the `~`-then-`?` route, and the Spanish-keyboard `¶` report)
- [AlteredGamer — Far Cry 2 devmode and cheats](https://www.alteredgamer.com/far-cry-2/22835-cheats-devmode-cheat-codes-and-shortcuts/)
  (the `-DEVMODE` launch parameter, documented as unlocking all levels)
- [GameFAQs — Far Cry 2 console commands and tweaks](https://gamefaqs.gamespot.com/boards/942192-far-cry-2/46145975)
- Cross-engine library: the console-key scancode trap and the control rules —
  [`docs/techniques/`](https://github.com/TefMeister/flat-to-vr-cross-engine-research/blob/main/docs/techniques/README.md)
