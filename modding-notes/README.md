# far-cry-2-vr — `modding-notes/`

Readable, chronological **field notes** from the Far Cry 2 VR reverse-engineering
project (2008, Ubisoft Montreal — **Dunia Engine**). This is the human-facing
progress ledger: what we tried, what worked, what didn't, and why — in plain
language, dated newest concerns last.

For the *distilled current truth* (the engine dossier), see
[`engine-research/`](../engine-research/).
For raw dumps and probe artifacts, see
[`dev-archive/`](../dev-archive/).
For releases (none yet), see
[`mod/`](../mod/).

## The folders for Far Cry 2 VR

Everything for this game lives in one repository, one folder per job — so you
always know where to look. You are in **`modding-notes/`**.

| Folder | What lives here |
| --- | --- |
| [`mod/`](../mod/) | The mod itself — the Far Cry 2 (Dunia engine) VR mod (pre-release; RE in progress). |
| [`dev-archive/`](../dev-archive/) | Full development history — snapshots, probes, dead ends, raw recon. |
| **`modding-notes/`** ← you are here | Readable field notes / progress ledger. |
| [staging/far-cry-2-vr](https://github.com/TefMeister/staging/tree/main/far-cry-2-vr) 🔒 | **Private** — unverified WIP builds, cross-machine handoff. |
| [`engine-research/`](../engine-research/) | Distilled engine reference (dossier) + reusable VR RE playbook. |
| [`external-research/`](../external-research/) | Ongoing public-research leads, gathered separately from hands-on modding work. |

## Progress log

- **[2026-08-21 — Phase 0 kickoff & static recon](2026-08-21-phase0-kickoff-and-recon.md)**
  — project set up, five repos created, first look at the binaries. Established
  that Far Cry 2 is a 32-bit D3D9 Dunia engine living in `Dunia.dll`, with a rich
  editor camera API that should shortcut finding the camera.
- **[2026-08-21 — Phase 1 foothold & camera struct](2026-08-21-phase1-foothold-and-camera.md)**
  — recovered the full camera struct statically (position/FOV/basis/angles via
  the `FCE_Camera_*` accessors), and built a compile-verified 32-bit `winmm.dll`
  proxy with a read-only camera probe. Not yet run in-game.
- **[2026-08-21 — First live debug: the editor API is a dead end](2026-08-21-phase1-live-debug-editor-api-dead-end.md)**
  — ran the proxy in-game (foothold works!) and attached a debugger: the FCE
  editor camera/globals are all null during real gameplay. The game camera is a
  different class. Next: hook Direct3D 9 to get the real camera.
- **[2026-08-21 — Found the game camera on the render path](2026-08-21-found-the-camera-on-the-render-path.md)**
  — hooked D3D9 `Present` + `SetVertexShaderConstantF`; the camera view matrix is
  at shader constant register c12 (exact inverse at c36), position confirmed by
  walking. The keystone for VR is in hand.

## Legal

Non-commercial fan modding notes. Requires a legitimately owned copy of Far Cry
2. No original game assets are included.

## Contributing & policy

See [CONTRIBUTING.md](CONTRIBUTING.md) — how we credit and link sources, our
**study-everything-public but write-our-own-code** rule (we copy no one else's
source code or files, any license or price), the terms for reusing our work
(free, with credit), and how to request a correction or removal.
