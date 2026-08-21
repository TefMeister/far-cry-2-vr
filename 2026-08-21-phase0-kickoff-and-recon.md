# 2026-08-21 — Phase 0: kickoff & static recon

## What happened today

Kicked off the **Far Cry 2 VR** project. Set up the standard five-repo layout
(`far-cry-2-vr-mod`, `-dev-archive`, `-modding-notes`, `-staging`,
`-engine-research`) and did a first, purely-static reconnaissance pass over the
shipped Steam binaries — no debugger, nothing injected, just reading the files on
disk.

## The headline facts

Far Cry 2 runs on **Dunia**, Ubisoft Montreal's in-house fork of the original Far
Cry / CryEngine. Nice bit of lineage: this is the *first* Dunia game, the
ancestor of the engine behind Far Cry 3/4/5.

The practical shape of the target:

- **It's a 32-bit engine, and it all lives in `Dunia.dll` (19 MB).** The
  `FarCry2.exe` you launch is a tiny 27 KB stub that just calls `RunGame()`
  inside Dunia and gets out of the way. So anything we build has to be **x86**,
  and Dunia is where all the reverse-engineering happens.
- **The renderer is Direct3D 9** by default (there's a D3D10 path too, but the
  profile on this machine is set to `d3d9`). That's good news for a first hook —
  D3D9 is a simpler, well-trodden surface than modern D3D11/12.
- `FC2.dll` turned out to be a red herring — it's an activation shim
  (`paul_dll_activate_and_play.dll`), not engine code. Noted so we don't waste
  time on it later.
- **No SecuROM / no Denuvo** in the single-player binary (Steam stripped it), and
  PunkBuster is multiplayer-only. So there's no DRM wall to fight for
  single-player injection. That's a relief.

## The pleasant surprise

`Dunia.dll` exports **404 named functions**, and a big chunk of them are the
**Far Cry 2 map editor's C-API** (`FCE_*`). Among them is a whole
**`FCE_Camera_*`** family — `GetPos/SetPos`, `GetAngles/SetAngles`, `Rotate`,
`GetFOV`, the front/right/up basis vectors, camera speed. These are editor entry
points, but they're thin wrappers over the *same* camera the game uses. That
should massively shortcut the usual worst part of a VR conversion — finding the
camera in memory — and might even give us a ready-made way to *drive* the camera
for an automated test harness. Big early lead; want to confirm it holds up once
we're actually in-process.

## What we deliberately did NOT do

No dynamic analysis yet. We haven't attached a debugger, haven't hooked
anything, haven't confirmed how the view/projection matrices actually reach the
GPU, and haven't checked whether the retail build has a usable dev console. All
of that is Phase 1+.

## Known risks I'm already thinking about

- **D3D9 → OpenXR is not free.** Modern VR runtimes are D3D11/12-first. Presenting
  a D3D9 frame to a headset will likely need a D3D9→D3D11 shared-texture bridge.
  That's the scariest architectural unknown; want to scope it early.
- **32-bit address space** — engine + a VR runtime in one ~2–3 GB usable process
  could get tight.

## Next session

1. Pick and stand up an injection foothold (leaning toward an `eax.dll` or
   `d3d9.dll` proxy — `eax.dll` is already a loose file in `bin\`).
2. Get our DLL loading and confirm a D3D9 `Present` hook fires.
3. Probe whether the `FCE_Camera_*` exports are callable/observable from inside
   the running game, and start locating the live camera struct.

Full technical detail is in the engine-research dossier and the dev-archive
recon folder.
