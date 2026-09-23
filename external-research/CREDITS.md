# Credits & Attribution

This project is a reverse-engineering and modding effort built on the public
research, tools, and creative work of many people who came before us. None of
this would be possible without them. We list every source, tool, and prior
work we have drawn on below — by name or handle, as accurately as we could
verify it — including those that helped only as inspiration.

If we have missed someone, the omission is a mistake, not a slight. Please see
the "Get credited, or ask us to stop" section at the bottom.

## The original game

Far Cry 2 (2008) is the creative work of its developer and publisher. We are
only modding it; we did not make it, and all rights to the game and its assets
belong to their owners. No game files are included in any repository in this
project.

| Work | Creator(s) | Note |
|---|---|---|
| Far Cry 2 (2008), original game | Ubisoft Montreal (developer); Ubisoft (publisher) | Open-world FPS. |
| Dunia Engine (the base engine) | Ubisoft Montreal | In-house fork of the original Far Cry / CryEngine codebase; ancestor of the Dunia 2 engine behind Far Cry 3/4/5. |

## Prior art, tools, and research this repo draws on

This is a new repo (seeded 2026-08-24) for public-research leads specifically —
see [far-cry-2-vr-modding-notes](https://github.com/TefMeister/far-cry-2-vr/tree/main/modding-notes)'s own
`CREDITS.md` for the full list of tools/prior-art the mod itself already draws on. This table
grows as `/game-research far-cry-2-vr` finds new leads.

| Source / Work | Creator(s) | Link |
|---|---|---|
| OpenVR (SteamVR AER/per-eye-pose bug research) | Valve / ValveSoftware GitHub issue reporters | https://github.com/ValveSoftware/openvr |
| Microsoft Learn (D3D9Ex/D3D11 surface-sharing & GetRenderTargetData documentation) | Microsoft | https://learn.microsoft.com |
| GameDev.net forum community (D3D9 readback & shared-surface technique discussions) | GameDev.net contributors | https://gamedev.net |
| R.E.A.L. VR mods (AER technique reference) | Luke Ross | https://www.patreon.com/realvr |
| VRto3D project (community documentation of R.E.A.L.'s AER render modes) | oneup03 and contributors | https://github.com/oneup03/VRto3D |
| VR FOV calculation reference | risa2000 | https://risa2000.github.io/vrdocs |
| Sensics (VR FOV conversion reference) | Sensics | https://medium.com/insights-on-virtual-reality |
| Vireio Perception / VRBoost (alternate head-tracking-injection technique, prior art) | cybereality and the Vireio Perception contributors | https://github.com/cybereality/Perception |
| Helix Mod: Far Cry 2 (DX9) — 3D Vision fix | Helix Mod community | https://helixmod.blogspot.com/2013/01/far-cry-2-dx9.html |
| SteamVR/Frame Timing documentation | Valve | https://developer.valvesoftware.com/wiki/SteamVR/Frame_Timing |
| Far Cry 2 Crew Steam group — the `devmodeon` console unlock and its misleading error message | Far Cry 2 Crew community members | [steamcommunity.com/groups/FarCry2Crew](https://steamcommunity.com/groups/FarCry2Crew/discussions/6/2269193447675170650/) |
| AnandTech forum thread documenting Far Cry 2's console commands and the `gfx_*` family | AnandTech forum members | [forums.anandtech.com](https://forums.anandtech.com/threads/far-cry-2-console-commands.230928/) |
| Far Cry 2 devmode / cheats guide (the `-DEVMODE` launch parameter) | AlteredGamer | [alteredgamer.com](https://www.alteredgamer.com/far-cry-2/22835-cheats-devmode-cheat-codes-and-shortcuts/) |
| Far Cry 2 console commands and tweaks board | GameFAQs community | [gamefaqs.gamespot.com](https://gamefaqs.gamespot.com/boards/942192-far-cry-2/46145975) |

AI development assistance: **Claude (Anthropic)** (https://www.anthropic.com).

Project lead and author: **TefMeister**.

## Missing from this list?

If you — or someone whose work you know — contributed to, influenced, or
even just inspired anything used in this project and you aren't credited
here, please **open a GitHub issue on this repo** and we'll correct it as
soon as possible. We would much rather over-credit than leave anyone out.

## Added by the 2026-09-11 research pass

For the culling and weapon-separation findings behind
`topics/2026-09-11-vorpx-says-the-matrix-cannot-fix-culling-and-the-weapon-has-a-fix-ladder.md`:

- **Ralf**, author of **vorpX**, and forum users **Eola667**, **CowPox** and **cercata** — for stating
  on the record that head rotation cannot be decoupled from the game camera in a FullVR style, for
  describing our exact "incomplete geometry or the void" symptom before we hit it, and for the Far Cry 2
  specific report that DirectVR works here while weapons are "not in scale".
- **DHR** — the HelixMod *Far Cry 2 (DX9)* 3D Vision fix (2013), including the `O`/`P` convergence
  presets for aiming, and **DarkStarSword** for the right-mouse-held improvement in its comments and for
  the `3d-fixes` toolchain.
- **bo3b (Bo3b Johnson)** — *School for Shaderhackers* and the 3Dmigoto wiki, for the canonical
  `w`-proportional stereo formula and `StereoParams`.
- **davegl1234** and the HelixMod community — *geo-11*, for the per-draw weapon convergence approach
  filtered by index-buffer and texture hash; binaries mirrored by **ThreeDeeJay**.
- **LukeRoss** — the *R.E.A.L.* mod, for the hybrid architecture (game-camera rotation plus a render-time
  view fix), the dominant-eye weapon alignment, dynamic crosshair depth, and the candid notes on FOV
  patching breaking streaming and on the game "wrestling for camera ownership".
- **NVIDIA GameWorks** — the `dxvk-remix` *Anti-Culling System* documentation, and its honest statement
  of what anti-culling cannot do.
- **praydog** — UEVR and its write-up, for the culling cvars and the `IStereoRendering` approach.
- **cybereality / Denis Reischl** and contributors — *Vireio Perception* and VRBoost's memory-write
  approach to camera rotation.
- **the opentrack contributors** — issues #113, #120 and #803, for the documented failure modes of
  mouse-emulated head tracking.
- **DR-89** — *fear-vr*; **BerZerker96** — the *6DOF Head-Tracking Mods Hub*; **fholger (Holger
  Frydrych)** — the Far Cry 1 VR mod.
- **FoxAhead** — *Far Cry 2 Multi Fixer*, for the runtime Dunia memory-patching precedent.
- **Crytek**, and **Carl Jones** and **Sean Patrick Tracy** (GDC Online 2010) — for CryEngine 3's
  near-geometry stereo handling, which shows a shipping engine choosing to move the weapon.
- **Flax Engine** documentation, and the Unreal and gamedev.net community threads on viewmodel FOV and
  Z-clipping.

## Respecting creators

This project exists because other people generously shared their
reverse-engineering research, tools, and modding know-how in public — we've
tried to credit every one of them by name or handle above, as accurately as
we could verify. If you are the creator or rightful owner of anything
credited or used here and you'd rather your work not be referenced in this
repo, or you want specific content removed or no longer used by the mod,
please tell us: **open a GitHub issue on this repo**. We'll act on that
request promptly — no argument, no delay — and we'll find another way to get
the job done that doesn't rely on your material. This is your work; we're
just grateful to have learned from it.

## Sources (2026-09-23)

- **phunkaeg**, VR Modding Playbook — https://github.com/phunkaeg/vr-modding-playbook
- **MonsterDeadWood**, FC2VR, as studied in the playbook
