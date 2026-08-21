# far-cry-2-vr-engine-research

Distilled, reusable reverse-engineering knowledge for the **Far Cry 2 VR**
project. This is the *current-truth* engine reference, kept separate from the
chronological history in the `-dev-archive` / `-modding-notes` repos.

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
