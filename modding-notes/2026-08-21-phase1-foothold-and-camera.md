# 2026-08-21 — Phase 1: the foothold, and finding the camera

Second session of the day, and a good one — two real milestones.

## 1. We found the camera (statically!)

The big worry in any VR conversion is "where is the camera in memory, and can we
read/override it?" Far Cry 2 handed us a shortcut. The map editor's C-API
(`FCE_Camera_GetPos`, `GetFOV`, `GetFrontVector`, …) are just thin accessors
over the engine's live camera object. So instead of hunting through a running
process, I disassembled those little functions and *read the struct off them*.

They all go through one global pointer (`Dunia.dll+0x0164FE7C`), and that pointer
is referenced **101 times** across the engine — so it's a central object, almost
certainly the actual game camera, not just an editor toy.

The object turns out to be a textbook camera:

- **position** at +0x0C / +0x10 / +0x14
- **FOV** at +0x2C
- a full **right / up / front basis** at +0x64 / +0x58 / +0x4C
- **euler angles** at +0x70 / +0x74 / +0x78

That's genuinely everything we need to read to build a view matrix, plus a FOV
knob for per-eye projection later. Really encouraging this early.

One caution I noted for future-me: the *setters* don't just poke the fields —
they route through a helper function that also talks to what looks like the
scene/collision manager (the editor's "clip camera to terrain" logic). So when
we get to *overriding* the camera, writing the struct directly might get ignored
or overwritten. We'll probably have to go through that helper or apply the
override at the GPU boundary instead. Noted, not yet tested.

## 2. We have a foothold that builds

I stood up the Phase-1 injection DLL — a 32-bit `winmm.dll` proxy (Dunia imports
winmm, and it's not a protected "KnownDLL", so our copy in `bin\` loads first).
It forwards all 180 real-winmm exports so the game is unaffected, writes a log to
`%LOCALAPPDATA%\FC2VR\fc2vr.log`, and — the useful part — runs a **read-only
camera probe**: once the engine's loaded it reads that camera pointer every
second and logs the position/FOV/front-vector.

That probe is deliberately the *first* thing we run, because it answers the one
open question the static work can't: **is that camera global actually live
during real gameplay?** When this gets a test run in the game, the log will tell
us straight away.

It **compiles cleanly to a 32-bit DLL** (verified: PE32/i386, 180 exports). It
has **not been run in the game yet** — that's the next step. Reusing the winmm
scaffold from the earlier projects needed a 32-bit fixup (the old one was
64-bit); the fiddly bit was assembly symbol decoration, now sorted and written
down so we don't rediscover it.

No hooks are installed yet — this build only reads memory. The Direct3D 9
Present/camera hooks come next, once the probe confirms the camera is what we
think it is.

## Where things stand

- Engine model: camera **located and mapped**; how the view/projection reaches
  the GPU is still unknown (next to investigate).
- Foothold: **built, not yet run.**

## Next session

1. Test-run the proxy in the game and read the log — does the banner appear, and
   is the camera pointer live with sane values during gameplay?
2. If yes: add the D3D9 `Present` hook (via a dummy-device vtable grab) and start
   inventorying render passes.
3. Work out whether view/projection arrives via `SetTransform` or
   `SetVertexShaderConstantF`.

Full technical detail is in the engine dossier (research repo) and the
dev-archive recon folder; the code is in the staging repo under `proxy-winmm/`.
