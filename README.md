# Far Cry 2 VR

A work-in-progress **VR support mod for Far Cry 2** (2008, Ubisoft Montreal —
**Dunia Engine**).

> **This is not playable yet.** There is no release. Until you see a tagged
> release here that explicitly says it runs in a VR headset, treat this as
> developer work-in-progress, not something to play.

**Status: pre-planning / reverse-engineering — not VR-ready.** No playable
release yet, and no mod code written yet. The project is currently at **Phase 0**
of the VR playbook: static reconnaissance of the engine. What we know so far —
Far Cry 2 runs a 32-bit **Direct3D 9** Dunia engine entirely inside `Dunia.dll`,
loaded by a thin `FarCry2.exe` launcher — is written up in the engine-research
repo. The near-term goal is the usual foundation: get our own code running in the
process, hook the renderer, and locate the camera matrices; stereo and head
tracking come after that.

This repository will hold **only files we create** for the mod. No game files
are included, and none ever will be — the mod requires you to own a legitimate
copy of Far Cry 2 and redistributes no original assets.

## The five repositories for Far Cry 2 VR

Everything for this game lives in five repositories, each with one job — so you
always know where to look. You are in **far-cry-2-vr-mod**.

| Repository | What lives here |
| --- | --- |
| **far-cry-2-vr-mod** ← you are here | The mod itself — the Far Cry 2 (Dunia engine) VR mod (pre-release; RE in progress). |
| [far-cry-2-vr-dev-archive](https://github.com/TefMeister/far-cry-2-vr-dev-archive) | Full development history — snapshots, probes, dead ends, raw recon. |
| [far-cry-2-vr-modding-notes](https://github.com/TefMeister/far-cry-2-vr-modding-notes) | Readable field notes / progress ledger. |
| [far-cry-2-vr-staging](https://github.com/TefMeister/far-cry-2-vr-staging) 🔒 | **Private** — unverified WIP builds, cross-machine handoff. |
| [far-cry-2-vr-engine-research](https://github.com/TefMeister/far-cry-2-vr-engine-research) | Distilled engine reference (dossier) + reusable VR RE playbook. |

## Credits

This mod stands on community reverse-engineering of the Far Cry / Dunia engine
and on open-source modding tools. Full credits will be maintained here as the
project takes shape. If you should be credited and aren't, email
**td3kxlvr@proton.me** and we'll fix it ASAP. We honour correction and removal
requests from the rights holders of anything used.

## Legal

Non-commercial fan mod. Requires a legitimately owned copy of Far Cry 2.
Redistributes no original game assets.

## Contributing & policy

See [CONTRIBUTING.md](CONTRIBUTING.md) — how we credit and link sources, our
**study-everything-public but write-our-own-code** rule (we copy no one else's
source code or files, any license or price), the terms for reusing our work
(free, with credit), and how to request a correction or removal.
