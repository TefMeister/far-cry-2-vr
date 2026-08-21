# Engine Dossier — Far Cry 2 (Dunia Engine, 2008)

> One consolidated, living reference for this game's engine, filled in as the
> `PLAYBOOK.md` phases are worked. Chronological blow-by-blow belongs in the
> `-dev-archive` / `-modding-notes` repos; this file is the *distilled current
> truth*. Update it whenever a fact changes; correct false leads in place.

**Status:** Phase 0 — Ground truth & setup (static recon only; nothing injected
yet) · **VR-readiness verdict:** TBD — feasible on paper (in-process x86 engine,
D3D9/D3D10 renderer, clean editor C-API surface), no camera work done yet.

---

## 1. Identity
- **Game / build / version:** Far Cry 2 (Fortune's Edition on this machine),
  Steam install. Engine binary reports internal version `0,1,0,1`; game shipped
  patch level **1.03** per `PatchNotes.txt` (v1.00 → 1.01 → 1.02 → 1.03
  cumulative).
- **Platform & store:** PC, Steam (`steamapps/common/Far Cry 2`). Native PC
  title, not a port. Steam build — **no SecuROM** strings present in `Dunia.dll`
  (0 hits), so the disc-era DRM was stripped for the digital release. One
  `uplay` string only (vestigial), so no live Ubisoft-services gate to fight for
  single-player injection.
- **Legitimacy:** owned copy confirmed (user's Steam library).

## 2. Engine lineage
- **Family / base engine:** **Dunia** — a heavily-forked branch of the original
  Far Cry 1 **CryEngine**, taken in-house by Ubisoft Montreal. This is the
  *first* Dunia game; Dunia 2 later powered Far Cry 3/4/5, so findings here are
  the genealogical root of the whole modern Far Cry renderer.
- **Middleware (string-count evidence in `Dunia.dll`):**
  - **Havok** — physics (`havok`, 25 hits).
  - **RealTree** — vegetation / foliage system (`RealTree`, 102 hits; also a
    `<RealTreeProfile>` block in `GamerProfile.xml`).
  - **amBX** — Philips ambient-lighting peripheral support (`ambx`, 53 hits;
    `UseAmbx` in profile). Irrelevant to VR but confirms era.
  - **EAX** audio (`eax.dll` present, imported by Dunia) + Bink video
    (`binkw32.dll`) for cutscenes.
  - **PunkBuster** anti-cheat (`pb/` folder, 15 hits) — **multiplayer only**;
    does not touch the single-player process we target.
  - No PhysX, no OpenAL, no FMOD (fmod = 1 spurious hit).
- **Distinctive file formats / build tags:**
  - **FAT/DAT archive pairs** in `Data_Win32/` (`common.fat`+`common.dat`,
    `patch.fat`+`patch.dat`, `shadersobj`, `sound*`) — Dunia's packed VFS.
    `patch.*` overlays the base archives (patch-over-base load order).
  - Config is **XML** (`Config/DefaultEngineConfig.xml`,
    `Config/ResourceConfig.xml`, `DefaultUserControls.xml`,
    `InputUserActionMap.xml`, `*.game.xml`, `*.meta.xml`, `map.xml`,
    `moviedata.xml`) — most live *inside* the archives, not loose on disk.
  - MSVC 2005 toolchain: imports `MSVCR80.dll` / `MSVCP80.dll`; C++ exports use
    classic MSVC name-mangling (`??0…@@QAE@XZ`).

## 3. Binary & memory
- **Bitness / layout:** **32-bit (x86 / `pe-i386`)** across the board —
  `FarCry2.exe`, `Dunia.dll`, `FC2.dll`. This is a 32-bit target; injected code
  and any VR runtime shim must be x86.
- **Module split:**
  - `FarCry2.exe` (27 KB) is a **thin launcher stub**. It imports exactly three
    Dunia exports and hands control over:
    - `?RunGame@@YA_NPAUHINSTANCE__@@PBD@Z` (ordinal 11) — `bool RunGame(HINSTANCE, const char*)`, the real entry point.
    - `RegisterGameFunctionProvider` (ord 393)
    - `AddFunctionCB` (ord 12)
    - Imports otherwise only `KERNEL32` + `MSVCR80`. So **all engine logic lives
      in `Dunia.dll`** (19.2 MB) — that is the reverse-engineering target.
  - `FC2.dll` (380 KB) is unrelated to the engine — its internal name is
    `paul_dll_activate_and_play.dll`, i.e. a **launcher/activation shim**, not
    engine code. Ignore for RE.
- **Renderer API (evidence):** **Direct3D 9 by default, with a Direct3D 10
  path.**
  - `Dunia.dll` statically imports **`d3d9.dll`** + **`d3dx9_38.dll`** (D3DX June
    2008).
  - It *also* references **`d3d10.dll`, `d3d10_1.dll`, `dxgi.dll`** as strings
    (dynamically loaded) and exports `FCB_Benchmark_IsD3D10Supported` — so DX10
    is a runtime-selected alternate path.
  - Active path on this machine is **D3D9**: `GamerProfile.xml` →
    `<RenderProfile Platform="d3d9" …>`. The `customd3d10` quality block shows
    DX10 unlocks HDR-FP32.
  - **VR implication:** hook **D3D9** first (matches the shipped default and is
    the simpler surface). `IDirect3DDevice9::Present` / `SetTransform` /
    `SetVertexShaderConstantF` are the likely camera-delivery hook points — TBD.
- **Developer console / cvar system:** engine is CryEngine-derived, so a cvar
  system almost certainly exists internally. Console-style tokens seen are mostly
  network `D_TRACE_*` trace flags, not render cvars. **Whether a console can be
  opened in the retail build is UNKNOWN** — needs testing (Phase 0/2). Render
  tuning is exposed via `GamerProfile.xml` (`WidescreenFOV`, `AspectRatio`,
  `ForceWidescreen`, MSAA, resolution) rather than a live console, as far as we
  know so far.

## 4. DRM / anti-debug & injection foothold
- **DRM:** none observed in the single-player binary (no SecuROM/Denuvo; Steam
  wrapper only). PunkBuster is MP-only.
- **Attach workflow:** not yet attempted. Plan: launch via Steam, attach x64dbg
  (32-bit engine → use x32dbg) to `FarCry2.exe`, but breakpoints of interest are
  in the loaded `Dunia.dll` module.
- **Injection vector (candidates, none tried yet):**
  - **Proxy DLL** — Dunia imports several hijackable system DLLs. Strong
    candidates: **`eax.dll`** (already a loose file in `bin/`, so a proxy drops
    in cleanly), `binkw32.dll`, or a `dinput8.dll`/`d3d9.dll` proxy. `eax.dll`
    is the cleanest since it's a small, non-critical, already-local dependency.
  - Alternatively a classic injector into `FarCry2.exe`.
  - **Decision: TBD** — lean toward `eax.dll` or `d3d9.dll` proxy.

## 5. Threading & frame structure
- Unknown / not yet investigated. CryEngine-1-era, so expect a **main thread +
  render thread** split but likely *immediate-mode* D3D9 (no deferred contexts /
  command lists — that's a D3D11 concept; D3D9 has none). One `RunGame` drives
  the loop.
- One-frame walkthrough: **TBD** (Phase 3).

## 6. Camera & projection delivery (the crucial section) — NOT YET DONE
- **Editor C-API hint:** `Dunia.dll` exports a rich **`FCE_Camera_*`** family
  (the Far Cry 2 map-editor's camera interface into the live engine) — 14
  functions incl. `FCE_Camera_GetPos/SetPos`, `GetAngles/SetAngles`, `Rotate`,
  `GetFOV`, `GetFront/Right/UpVector`, `Get/SetSpeed`. These are *editor*
  entry points, but they are thin wrappers over the **same underlying camera
  object** the game uses, so they're a gift: they name the fields and give us
  ready-made read/write hooks to locate the camera struct in memory.
  `FCE_Engine_UpdateViewport` is the likely per-frame viewport push.
- **How the world transform reaches the GPU:** UNKNOWN. D3D9 means it's *either*
  fixed-function `SetTransform(WORLD/VIEW/PROJECTION)` *or* (far more likely for
  a 2008 shader engine) **vertex-shader constants via
  `SetVertexShaderConstantF`** — a shared VP matrix in a known constant register
  range. Must confirm by hooking + shader disassembly (`shadersobj.dat`).
- **CB slot / offset / handedness / FOV source / per-eye maths:** all **TBD**
  (Phase 4 keystone).

## 7. Constant-buffer fill mechanism — NOT YET DONE
- D3D9 has no `Map`/`UpdateSubresource` constant-buffer model; matrices are
  pushed each frame via `SetVertexShaderConstantF` (float register file) or
  `SetTransform`. The override point is therefore an **API-call hook**, not a
  memory-mapped ring. Confirm which call carries VIEW/PROJ. **TBD.**

## 8. Pass inventory (by render target) — NOT YET DONE
- `shadersobj.dat` (249 MB) holds compiled shaders — the pass structure is in
  there. Deferred? FC2 is known to use a **light-pre-pass / deferred-ish**
  renderer with dynamic weather, day/night, and the signature fire propagation.
  Main scene, shadow maps, post/HDR/bloom (per `customd3d10` profile), HUD —
  all **TBD**.

## 9. cvar / console cheat sheet
| command / cvar | effect | use |
|---|---|---|
| `GamerProfile.xml` `WidescreenFOV="1"` | enables wider FOV scaling | possible stopgap FOV lever before real per-eye proj |
| `GamerProfile.xml` `ForceWidescreen` / `AspectRatio` | aspect handling | needed to drive an ultra-wide / per-eye aspect |
| `GamerProfile.xml` `Platform="d3d9"\|"dx10"` | selects renderer | keep `d3d9` for first hook target |
| in-game console | **existence UNCONFIRMED** | investigate Phase 0/2 |

## 10. Autonomous harness recipe (this game) — NOT YET BUILT
- Launch to a known scene: TBD (Steam launch; skip intro; load a save).
- In-process input / camera drive: the **`FCE_Camera_Set*`** exports are a
  promising in-process drive method for a test harness even outside the editor —
  worth probing early.
- Frame-capture method: TBD (hook Present → dump backbuffer).

## 11. Dead ends & false leads (save future time)
- **`FC2.dll` is NOT engine code** — it's `paul_dll_activate_and_play.dll`, an
  activation shim. Don't waste time disassembling it for engine internals.
- **`FarCry2.exe` is a 27 KB stub** — don't look for logic there; everything is
  in `Dunia.dll`.
- **PunkBuster / `pb/`** is multiplayer-only; not a single-player anti-tamper
  obstacle.

## 12. Open risks toward the North Star
- **32-bit address space** — a stereo/VR runtime (OpenXR/OpenVR) plus the engine
  in a 4 GB (really ~2–3 GB usable) x86 process could get memory-tight; watch
  allocations.
- **D3D9 + OpenXR** — modern XR runtimes are D3D11/12-first; presenting a D3D9
  swapchain to an XR compositor typically needs a **D3D9→D3D11 shared-texture
  bridge**. This is the single biggest architectural risk and should be
  scoped early.
- **Deferred/HDR renderer** may make clean per-eye camera override harder than a
  forward renderer, if lighting passes bake in view-space assumptions.
- **Console existence unconfirmed** — if there's no dev console, all tuning is
  via XML + hooks, raising harness effort.

---

*Recon method: static PE analysis (`objdump`, `pefile`) + string mining of
`Dunia.dll`, plus `GamerProfile.xml` inspection. No dynamic analysis performed
yet. Full 404-export dump in [`dunia-exports.txt`](dunia-exports.txt).*
