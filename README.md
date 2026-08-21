# far-cry-2-vr-dev-archive

The full, messy **development archive** for the Far Cry 2 VR mod — snapshots,
probes, raw reconnaissance data, disassembly scratch, and the in-progress
history behind the mod. Things here are *not* guaranteed correct or final; the
distilled current truth lives in
[far-cry-2-vr-engine-research](https://github.com/TefMeister/far-cry-2-vr-engine-research),
and readable field notes in
[far-cry-2-vr-modding-notes](https://github.com/TefMeister/far-cry-2-vr-modding-notes).

This repository contains **only files we create** during development. **No
original game files or assets are included** — only recon output, generated
interface metadata, and our own code/notes.

## Contents so far

- **`recon/2026-08-21-phase0-static/`** — first static-analysis pass over the
  shipped binaries (PE headers, imports/exports, middleware string mining,
  render config). Method notes + machine-readable dumps.
- **`recon/2026-08-21-phase1-camera-struct/`** — disassembly of the
  `FCE_Camera_*` exports that recovered the engine camera struct (position, FOV,
  basis vectors, angles) and its singleton global. Findings write-up + raw
  capstone disassembly.

## Legal

Non-commercial fan reverse-engineering for a personal VR mod; requires a
legitimately owned copy of Far Cry 2. No original assets redistributed.
