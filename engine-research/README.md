# far-cry-2-vr-engine-research

Distilled, reusable reverse-engineering knowledge for the **Far Cry 2 VR**
project. This is the *current-truth* engine reference, kept separate from the
chronological history in the `-dev-archive` / `-modding-notes` repos.

## The six repositories for Far Cry 2 VR

Everything for this game lives in six repositories, each with one job — so you
always know where to look. You are in **far-cry-2-vr-engine-research**.

| Repository | What lives here |
| --- | --- |
| [far-cry-2-vr-mod](https://github.com/TefMeister/far-cry-2-vr-mod) | The mod itself — the Far Cry 2 (Dunia engine) VR mod (pre-release; RE in progress). |
| [far-cry-2-vr-dev-archive](https://github.com/TefMeister/far-cry-2-vr-dev-archive) | Full development history — snapshots, probes, dead ends, raw recon. |
| [far-cry-2-vr-modding-notes](https://github.com/TefMeister/far-cry-2-vr-modding-notes) | Readable field notes / progress ledger. |
| [far-cry-2-vr-staging](https://github.com/TefMeister/far-cry-2-vr-staging) 🔒 | **Private** — unverified WIP builds, cross-machine handoff. |
| **far-cry-2-vr-engine-research** ← you are here | Distilled engine reference (dossier) + reusable VR RE playbook. |
| [far-cry-2-vr-external-research](https://github.com/TefMeister/far-cry-2-vr-external-research) | Ongoing public-research leads, gathered separately from hands-on modding work. |

## Contents

- **[PLAYBOOK.md](PLAYBOOK.md)** — the engine-agnostic VR reverse-engineering
  playbook (North Star = the game running in a headset with head tracking).
  Copied verbatim into every game project; not specific to Far Cry 2.
- **[ENGINE-DOSSIER.md](ENGINE-DOSSIER.md)** — the consolidated, living Far Cry 2
  / **Dunia Engine** dossier: renderer, binary layout, injection foothold,
  camera-delivery leads, middleware, dead ends, and open VR risks. This is where
  the game-specific truth accumulates.
- **[dunia-exports.txt](dunia-exports.txt)** — full dump of all 404 named
  exports from `Dunia.dll` (ordinal · RVA-based VA · mangled name). Interface
  metadata we generated; no game content.
- **[templates/](templates/)** — the per-engine research template the dossier
  was seeded from.

## Engine at a glance

Far Cry 2 (2008) runs on **Dunia**, Ubisoft Montreal's in-house fork of the
original Far Cry / CryEngine — the ancestor of the Dunia 2 engine behind Far Cry
3/4/5. The shipped renderer is **Direct3D 9** (with a selectable D3D10 path), and
the whole engine is a **32-bit `Dunia.dll`** loaded by a thin `FarCry2.exe` stub.
See the dossier for the evidence trail.

## Status

**Phase 0 — Ground truth & setup.** Static recon only; nothing injected yet.
VR-readiness verdict: feasible on paper, no camera work done. See the dossier's
status line for the live verdict.

## Credits & corrections

This project builds on community reverse-engineering knowledge of the Far Cry /
Dunia engine. If you contributed knowledge, tooling, or inspiration and aren't
credited, email **td3kxlvr@proton.me** and we'll fix it promptly. We honour
correction and removal requests from the rights holders of anything referenced
here.

## Legal

Non-commercial fan reverse-engineering for a personal VR mod. Requires a
legitimately owned copy of Far Cry 2. **No original game files or assets are
included in this repository** — only interface metadata and notes we generated
ourselves.

## Contributing & policy

See [CONTRIBUTING.md](CONTRIBUTING.md) — how we credit and link sources, our
**study-everything-public but write-our-own-code** rule (we copy no one else's
source code or files, any license or price), the terms for reusing our work
(free, with credit), and how to request a correction or removal.
