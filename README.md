# Far Cry 2 VR

A work-in-progress **VR support mod for Far Cry 2** (2008, Ubisoft Montreal —
**Dunia Engine**), aiming at full stereo rendering with head tracking in a PC VR
headset via SteamVR.

## ⚠️ What this mod is — and what it is not

**What it is (v0.1.x):** a developer alpha that proves the two hard parts work:

- **We control the game camera.** The mod intercepts the engine's camera
  matrices on the render path and can move the eye point per frame (visible as
  stereo "wiggle" test modes on the flat monitor).
- **The game reaches SteamVR.** The mod captures each rendered frame and
  submits it to the SteamVR compositor from inside the 32-bit game process, so
  the game screen appears in a connected headset.

**What it is NOT (yet):**

- It is **not stereoscopic 3D in the headset** — the headset currently shows
  the same flat image to both eyes (mono), stretched across each eye's view.
- It has **no head tracking** — moving your head does not move the in-game
  camera yet.
- It is **not a finished, playable VR experience**, and it does not change the
  game's controls, HUD, or comfort options in any way.

## 🤢 Caution: unfinished — may cause severe motion sickness

This mod is **unfinished** and in active development. Using it in a headset may
cause **severe motion sickness and discomfort**: the image is un-tracked (it
does not respond to your head), monoscopic, and stretched. Try it seated, take
it off at the first sign of discomfort, and treat every release as an
experiment until this caution is removed.

## Requirements

- A legitimate copy of **Far Cry 2** (tested: Steam build, patch 1.03).
  This mod contains and redistributes **no game files**.
- **SteamVR** and a PC VR headset (tested target: Quest 3 via Virtual Desktop).
- Windows 10/11.

## Install

1. Download the latest release ZIP from this repository's Releases page.
2. Copy `winmm.dll` and `openvr_api.dll` into the game's `bin` folder
   (next to `FarCry2.exe`), e.g.
   `...\Steam\steamapps\common\Far Cry 2\bin\`.
3. Start SteamVR.
4. **Launch `bin\FarCry2.exe` directly** (not through the Steam library — with
   SteamVR running, Steam's Desktop Game Theatre can hang the launch on the
   "Launching…" popup).
5. In the main menu or in game, press **F10**.

**Uninstall:** delete the two DLLs from the `bin` folder. The game is untouched
otherwise.

## Hotkeys

| Key | Action |
|---|---|
| **F10** | Toggle the SteamVR bridge (game screen in the headset) |
| **F5** | Toggle the stereo camera override (flat-monitor test) |
| **F6 / F7** | Decrease / increase eye separation |
| **F8** | Cycle eye mode: wiggle → left → right |
| **F9** | Experimental: also offset rigid view matrices |

Diagnostics are logged to `%LOCALAPPDATA%\FC2VR\fc2vr.log`.

## How it works (short version)

The mod is a `winmm.dll` proxy that loads inside the game, hooks the Direct3D 9
device, classifies the engine's shader-constant matrix uploads to find the
real camera every frame, and rewrites those uploads to move the eye. For the
headset path it copies the rendered backbuffer into a Direct3D 11 texture and
submits it to the SteamVR compositor through the OpenVR API. Full technical
detail lives in the sibling repositories below.

## The six repositories for Far Cry 2 VR

Everything for this game lives in six repositories, each with one job — so you
always know where to look. You are in **far-cry-2-vr-mod**.

| Repository | What lives here |
| --- | --- |
| **far-cry-2-vr-mod** ← you are here | The mod itself — releases and install instructions. |
| [far-cry-2-vr-dev-archive](https://github.com/TefMeister/far-cry-2-vr-dev-archive) | Full development history — snapshots, probes, dead ends, raw recon. |
| [far-cry-2-vr-modding-notes](https://github.com/TefMeister/far-cry-2-vr-modding-notes) | Readable field notes / progress ledger. |
| [far-cry-2-vr-staging](https://github.com/TefMeister/far-cry-2-vr-staging) 🔒 | **Private** — unverified WIP builds, cross-machine handoff. |
| [far-cry-2-vr-engine-research](https://github.com/TefMeister/far-cry-2-vr-engine-research) | Distilled engine reference (dossier) + reusable VR RE playbook. |
| [far-cry-2-vr-external-research](https://github.com/TefMeister/far-cry-2-vr-external-research) | Ongoing public-research leads, gathered separately from hands-on modding work. |

## Third-party components

- **MinHook** by Tsuda Kageyu and contributors (BSD-2-Clause) — function hooking.
- **OpenVR API** by Valve (BSD-3-Clause) — SteamVR interface; `openvr_api.dll`
  is redistributed in releases under its license.

License texts are included with every release. Everything **we** wrote is free
to use and modify with credit — see [CONTRIBUTING.md](CONTRIBUTING.md).

## Credits

See [CREDITS.md](CREDITS.md) — this project stands on community
reverse-engineering and open-source tools, and we credit everyone, including
inspirations. If you should be credited and aren't, open a GitHub issue and we
will fix it as soon as possible.

## Contributing & policy

See [CONTRIBUTING.md](CONTRIBUTING.md) for the project's rules: study-not-copy,
no game assets ever, credit everyone, and a no-questions removal policy for
rights holders.

## Legal

This is a non-commercial fan mod. It requires owning a legitimate copy of
Far Cry 2, includes no original game assets, and redistributes none. All rights
to Far Cry 2 and the Dunia Engine belong to Ubisoft.
