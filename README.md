# far-cry-2-vr-modding-notes

Readable, chronological **field notes** from the Far Cry 2 VR reverse-engineering
project (2008, Ubisoft Montreal — **Dunia Engine**). This is the human-facing
progress ledger: what we tried, what worked, what didn't, and why — in plain
language, dated newest concerns last.

For the *distilled current truth* (the engine dossier), see
[far-cry-2-vr-engine-research](https://github.com/TefMeister/far-cry-2-vr-engine-research).
For raw dumps and probe artifacts, see
[far-cry-2-vr-dev-archive](https://github.com/TefMeister/far-cry-2-vr-dev-archive).
For releases (none yet), see
[far-cry-2-vr-mod](https://github.com/TefMeister/far-cry-2-vr-mod).

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

## Legal

Non-commercial fan modding notes. Requires a legitimately owned copy of Far Cry
2. No original game assets are included.
