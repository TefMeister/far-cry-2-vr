# far-cry-2-vr — `dev-archive/`

The full, messy **development archive** for the Far Cry 2 VR mod — snapshots,
probes, raw reconnaissance data, disassembly scratch, and the in-progress
history behind the mod. Things here are *not* guaranteed correct or final; the
distilled current truth lives in
[`engine-research/`](../engine-research/),
and readable field notes in
[`modding-notes/`](../modding-notes/).

This repository contains **only files we create** during development. **No
original game files or assets are included** — only recon output, generated
interface metadata, and our own code/notes.

## The folders for Far Cry 2 VR

Everything for this game lives in one repository, one folder per job — so you
always know where to look. You are in **`dev-archive/`**.

| Folder | What lives here |
| --- | --- |
| [`mod/`](../mod/) | The mod itself — the Far Cry 2 (Dunia engine) VR mod (pre-release; RE in progress). |
| **`dev-archive/`** ← you are here | Full development history — snapshots, probes, dead ends, raw recon. |
| [`modding-notes/`](../modding-notes/) | Readable field notes / progress ledger. |
| [staging/far-cry-2-vr](https://github.com/TefMeister/staging/tree/main/far-cry-2-vr) 🔒 | **Private** — unverified WIP builds, cross-machine handoff. |
| [`engine-research/`](../engine-research/) | Distilled engine reference (dossier) + reusable VR RE playbook. |
| [`external-research/`](../external-research/) | Ongoing public-research leads, gathered separately from hands-on modding work. |

## Contents so far

- **`recon/2026-08-21-phase0-static/`** — first static-analysis pass over the
  shipped binaries (PE headers, imports/exports, middleware string mining,
  render config). Method notes + machine-readable dumps.
- **`recon/2026-08-21-phase1-camera-struct/`** — disassembly of the
  `FCE_Camera_*` exports that recovered the engine camera struct (position, FOV,
  basis vectors, angles) and its singleton global. Findings write-up + raw
  capstone disassembly.
- **`recon/2026-08-21-phase1-live-debug/`** — first live run + x32dbg session.
  Confirmed the FCE editor globals are all null during retail gameplay and the
  game camera is a different class (720 MB snapshot scan found no editor-layout
  match). Includes the debugging workflow gotchas learned.

## Legal

Non-commercial fan reverse-engineering for a personal VR mod; requires a
legitimately owned copy of Far Cry 2. No original assets redistributed.

## Contributing & policy

See [CONTRIBUTING.md](CONTRIBUTING.md) — how we credit and link sources, our
**study-everything-public but write-our-own-code** rule (we copy no one else's
source code or files, any license or price), the terms for reusing our work
(free, with credit), and how to request a correction or removal.
