# Engine Dossier — Far Cry 2 (Dunia Engine, 2008)

> One consolidated, living reference for this game's engine, filled in as the
> `PLAYBOOK.md` phases are worked. Chronological blow-by-blow belongs in the
> `-dev-archive` / `-modding-notes` repos; this file is the *distilled current
> truth*. Update it whenever a fact changes; correct false leads in place.

**Status:** **GAME CAMERA FOUND on the render path** (verified in gameplay
2026-08-21). D3D9 Present + `SetVertexShaderConstantF` hooks work; the camera
view matrix is uploaded to vertex-shader constant register **c12** (world→view)
with its exact inverse at **c36** (view→world), and the translation **tracks the
player as they walk** — confirmed. · **VR-readiness verdict:** feasible, on
track — we can read the live camera every frame. Next: derive/confirm the
projection and per-eye override maths.

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
- **Developer console — CONFIRMED to exist.** The export
  `FCE_Engine_IsConsoleOpen` reads a **console/UI-manager global at `0x11606280`**
  (RVA `0x1606280`) and returns the **byte at `+0x69`** as the "console open"
  flag. So a console subsystem is present with a live open/closed state — a
  strong lever for the Phase-2 autonomous harness (whether it can be *opened* in
  retail, and what commands it takes, is the next thing to probe). Other
  console-ish tokens in strings are mostly network `D_TRACE_*` flags. Render
  tuning is also exposed via `GamerProfile.xml` (`WidescreenFOV`, `AspectRatio`,
  `ForceWidescreen`, MSAA, resolution).

## 4. DRM / anti-debug & injection foothold
- **DRM:** none observed in the single-player binary (no SecuROM/Denuvo; Steam
  wrapper only). PunkBuster is MP-only.
- **Attach workflow:** not yet attempted. Plan: launch via Steam, attach x64dbg
  (32-bit engine → use x32dbg) to `FarCry2.exe`, but breakpoints of interest are
  in the loaded `Dunia.dll` module.
- **Injection vector — CHOSEN: `winmm.dll` proxy.** `Dunia.dll` imports
  `WINMM.dll`, and `winmm` is not a Windows *KnownDLL*, so a local copy beside
  `FarCry2.exe` wins the loader search. A 32-bit proxy forwarding all 180 real
  winmm exports (resolved at runtime from System32; the real DLL is never
  shipped) builds and is compile-verified in `-staging/proxy-winmm/`. Chosen over
  `eax.dll` for consistency with the proven cross-project scaffold and because it
  needs no rename dance with an already-local file.
  - Also viable, not needed: `eax.dll` (already loose in `bin/`), `d3d9.dll`,
    `dinput8.dll` proxies, or a classic injector into `FarCry2.exe`.

## 5. Threading & frame structure
- Unknown / not yet investigated. CryEngine-1-era, so expect a **main thread +
  render thread** split but likely *immediate-mode* D3D9 (no deferred contexts /
  command lists — that's a D3D11 concept; D3D9 has none). One `RunGame` drives
  the loop.
- One-frame walkthrough: **TBD** (Phase 3).

## 6. Camera & projection delivery (the crucial section) — CAMERA STRUCT FOUND (static)
- **Camera singleton located.** Disassembling the editor's `FCE_Camera_*`
  accessors (they are thin readers/writers over the engine's live camera object)
  revealed a global pointer and a fully-mapped struct. All 14 accessors read
  through the **same global**, and it has **101 xrefs** across `.text` — this is
  a core engine object, not an editor-only artefact.

  ```
  Dunia.dll preferred ImageBase : 0x10000000
  camera-singleton global (VA)  : 0x1164FE7C   →  RVA 0x0164FE7C
      CCamera* cam = *(CCamera**)(dunia_base + 0x0164FE7C);
  ```

  **Struct layout (byte offsets from `cam`), from disassembly:**

  | offset | field | source accessor |
  |---|---|---|
  | +0x0C / +0x10 / +0x14 | position x/y/z | `FCE_Camera_GetPos` |
  | +0x2C | FOV (units TBD: deg or rad) | `FCE_Camera_GetFOV` |
  | +0x4C / +0x50 / +0x54 | front (forward) vector | `FCE_Camera_GetFrontVector` |
  | +0x58 / +0x5C / +0x60 | up vector | `FCE_Camera_GetUpVector` |
  | +0x64 / +0x68 / +0x6C | right vector | `FCE_Camera_GetRightVector` |
  | +0x70 / +0x74 / +0x78 | euler angles (pitch/yaw/roll?) | `FCE_Camera_GetAngles` |

  A position + orthonormal right/up/front basis + FOV + euler angles: a textbook
  camera. The basis vectors give us the **view rotation directly** (no matrix
  decompose needed), and **FOV at +0x2C is the projection lever** for per-eye
  work. **Reads only are proven; ASLR means the global must be resolved as
  `GetModuleHandleW("Dunia.dll") + 0x0164FE7C` at runtime, never hardcoded.**
- **Writes are non-trivial.** `SetPos`/`SetAngles`/`Rotate` don't poke fields;
  they funnel through a helper at `0x10837A10` (which itself touches another
  global `0x1164FE04`, likely the scene/collision manager — consistent with the
  editor's "clip camera to terrain" option). So *overriding* the camera by
  writing the struct may need to go through, or after, that helper — or be
  applied downstream at the GPU boundary instead. TBD which.
- **ANSWERED (2026-08-21 live debug):** NO. Read via x32dbg in-game,
  `*(void**)0x1164FE7C == 0x00000000`. The editor camera global is **null during
  retail gameplay** — it is populated only when the map editor runs. The other
  `FCE_*` globals are unset too: console-mgr `0x11606280` and scene `0x1164FE04`
  read as null/garbage, and view `0x11609560` holds a stale pointer into a string
  constant. **The entire FCE editor API is editor-only.** Furthermore, a scan of
  a full 720 MB in-game memory snapshot for this exact struct layout (orthonormal
  basis at +0x4C + FOV at +0x2C + finite world position) found **no real camera**
  — so the game camera is a *different class* with a different layout. The struct
  above remains valid for the *editor* camera only.
- **View / viewport object located.** `FCE_Engine_UpdateViewport` writes width
  and height into a **global view object at `0x11609560`** (RVA `0x1609560`) at
  **`+0x20` (width)** and **`+0x24` (height)**, then calls a
  **viewport-change dispatcher at `0x103f8ab0`**. That dispatcher is a pub/sub
  fan-out: it walks several listener arrays (element stride `0x14`) and calls
  `vtable+4` on each registered observer — i.e. it *notifies* subsystems that the
  viewport changed. **The projection-matrix rebuild therefore lives inside one of
  those observer callbacks, not in this routine.** Finding that callback (and the
  camera→view→projection handoff) is far cheaper dynamically, once we're hooked,
  than by chasing vtables statically — so it's deferred to the Present-hook work.
- **Render device captured (2026-08-21).** The D3D9 `Present` hook
  (`-staging/proxy-winmm/src/d3d9_hook.c`) fires on the game's real device
  (`IDirect3DDevice9` @ e.g. `0x039757E0`, a heap COM object — not any FCE
  global). Confirmed facts from the hook:
  - **Back buffer: 1280×720, format 21 = `D3DFMT_A8R8G8B8`, no MSAA** (the
    game's real render-target size/format — needed for the VR path).
  - **`behaviorFlags = 0x40` = `D3DCREATE_HARDWARE_VERTEXPROCESSING`** → the game
    uses hardware vertex processing / programmable shaders, so the world
    transform almost certainly reaches the GPU via **`SetVertexShaderConstantF`**
    (a shared view-projection matrix in a constant-register range), NOT
    fixed-function `SetTransform`. This is the next hook to add.
  - Hook is stable: frame heartbeat climbs (#1 → #1024+) with no crash/perf hit,
    and it all fires at the **menu** (no gameplay/mouse needed) since Present
    runs every frame.
- **How the world transform reaches the GPU — SOLVED (2026-08-21).** Confirmed
  `SetVertexShaderConstantF`. Hooking it and dumping the low constant registers
  (first-write per frame, since the HUD overwrites low registers just before
  Present) located the camera:
  - **c12–c15 = VIEW matrix (world→view):** a rigid transform (orthonormal 3×3
    rotation + translation), row-vector/row-major (`m[3]=(0,0,0,1)`).
  - **c36–c39 = its exact inverse (view→world = camera world transform):**
    verified `c36 == inverse(c12)` (rotation transposed, translation `−RᵀT`).
    Its translation is the **camera world position**, and it **changed as the
    player walked** (X/Y move on a ~64-unit grid, Z≈1000 = eye height) — proof
    it's the camera, not a static object. Example (standing): pos ≈
    (2144, 2720, 1000), facing an axis (clean 90°-about-Z rotation).
  - **c16–c19 = projection**, but split with a **1/32 ↔ 32 scale** against the
    view (c0/c8 carry ~0.031 = 1/32 factors) — a float-precision trick for FC2's
    huge world. This is why a naive "sparse projection" heuristic does NOT match
    a clean D3D perspective matrix here (it false-flagged a swizzle at c71).
  - The game uploads **combined/scaled matrices**, not a standalone clean
    projection — plan per-eye maths around the c12 view + c16 scaled projection.
  - Caveat: the first-write-per-frame value may be a sector-relative view (the
    64-unit grid stepping); confirm whether the main-pass camera write differs
    when building the override. Full dump:
    `-dev-archive/recon/2026-08-21-phase1-live-debug/d3d9-vs-constants-gameplay-camera-found.log`.
- **Angle↔basis math exists:** `FCE_Core_GetAxisFromAngles` /
  `GetAnglesFromAxis` / `GetAxisFromAngles` build the rotation basis from euler
  angles (call into helper `0x10050990`) — the reference for handedness/rotation
  convention when we build per-eye view matrices.
- **CB slot / offset / handedness / FOV units / per-eye maths:** **TBD**
  (Phase 4 keystone).

## 6a. Head tracking: HMD pose -> the view-projection (built 2026-09-01, NEVER RUN)

`[verified-numerically 2026-09-01, n=300 synthetic cases]` for the maths;
`[untested]` for everything about the running game. No headset on the dev PC.

The composition never splits `M` into `P` and `V`, the same way the eye offset already avoided it.
Every factor sits on the RIGHT of `M`, i.e. in world space:

```
M' = M * [ R | c - R*(c + t) ]
```

### The basis is derived, not hardcoded

External research established that no published source gives a verified OpenVR->Dunia axis matrix,
that every real implementation hand-derives one, and that a wrong sign is invisible until someone
is wearing the headset. So nothing is assumed on either side:

* `row0.xyz` -> camera right, `row1.xyz` -> camera up, `row3.xyz` -> the view-depth axis. (`row3`
  is safe to use because `is_perspective_vp()` already requires it to be a roughly unit world-space
  axis - that classifier is what this rests on.)
* The single genuine convention difference - OpenVR's `+Z` points backwards where the game's third
  axis points forwards - is folded into the basis by negating its forward column. The whole
  conversion is then one change of basis, `R_camera = B*H*B^T`. **No permutation table exists in
  the code.**

### The camera position is SOLVED from M

For `M = P*V` the camera maps to the view origin, so three rows must vanish there:

```
row0.xyz . c + m[3]  = 0
row1.xyz . c + m[7]  = 0
row3.xyz . c + m[15] = 0
```

Three equations, three unknowns, by Cramer. **Row 2 is deliberately unused** - it carries the
near/far mapping and is the row most likely to be unusual in a given projection.

### Two traps, both found by numerical testing rather than by review

1. **Solve with the RAW rows, never the normalised basis.** `P`'s per-axis scales appear in the row
   *and* in that row's own translation term, so they cancel only if both come from the same
   unnormalised row. Mixing them silently scales the recovered camera position - the head then
   pivots about the wrong point, which is sickening rather than obviously broken.
2. **What post-multiplies into `M` is the INVERSE of the camera rotation.** `B*H*B^T` rotates the
   camera; the world transform is its transpose. Composing the camera rotation directly turns the
   view the right amount in the **wrong direction** - which in a headset presents exactly like a
   handedness problem, so the handedness knob would "fix" it and hide the real error.

`proxy-winmm/tools/verify_head_tracking.py` transcribes the algorithm and checks it against an
independently rebuilt `P*V'` for a camera actually rotated and translated. After the fixes,
agreement is ~1e-15; before, the relative error was **3.4**.

**Stronger check, added on review the same day:**
`verify_head_tracking_harness.c` compiles the **real `stereo.c`** (via `#include`, which also reaches
its statics) and drives `apply_head_tracking()` directly, with
`verify_head_tracking_against_c.py` comparing it to the same ground truth — **worst relative error
3.5e-07 over 200 cases, i.e. float32 precision.** The Python check proves the *algorithm*; this one
proves the *shipped code*. **Re-run the C one after any change here** - this is maths that cannot be
checked cheaply in a headset, and a transcription slip is the same family of error as the two bugs
above.

### Live knobs (all default OFF; require the VR bridge on Num0 for a pose)

`Num7` head tracking on/off - `Num2` flip rotation handedness - `Num1`/`Num3` position scale.

Dunia units are ~metres (2026-08-21), so scale 1.0 is an honest starting guess, not a measurement.

### The diagnostic that says the derivation itself is wrong

The periodic log reports `camera-position solve FAILED on N`. A few during a load is expected. **A
steady non-zero count means `row3` is not the forward axis for some perspective pass this game
makes, and the derivation needs revisiting - not a knob tuned.**

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
- **Don't hardcode `0x1164FE7C`** for the camera global — that's the VA at the
  preferred base; under ASLR you must use RVA `0x0164FE7C` + runtime module base.
- **Camera writes aren't field pokes** — `SetPos`/`SetAngles`/`Rotate` go through
  helper `0x10837A10`; naively writing the position floats may be ignored or
  overwritten. (Untested — flagged, not confirmed.)
- **32-bit proxy build gotcha (solved):** the shared winmm forwarder scaffold
  from prior projects is 64-bit (`jmp *ptr(%rip)`, undecorated symbols). For
  i686 it needs underscore-decorated asm labels, plain `jmp *_ptr_NAME`, and
  **bare** names in the `.def` (the i386 linker auto-prepends the underscore;
  writing `NAME = _NAME` double-underscores and fails to link).
- **The whole `FCE_*` (Far Cry Editor) export family is EDITOR-ONLY** — its
  globals are null in the retail game. Useful for recovering the *editor* camera
  struct layout and math conventions statically, but do NOT expect any FCE global
  to be live during gameplay. The game camera must come from the render path.
- **Debugging FC2 under x64dbg — two traps (both solved):**
  1. Launching FC2 *under* x32dbg and later detaching leaves DirectInput broken
     (mouse dead in-game). **Always launch the game normally and *attach* to the
     running process**; never launch-under-debugger then detach.
  2. Dunia names every thread via `MS_VC_EXCEPTION (0x406D1388)`, so x32dbg with
     default first-chance breaking stops constantly during init/level-load.
     Disable `[Events] DllEntry/TlsCallbacks/DllLoad/EntryBreakpoint/ThreadEntry`
     and/or add a first-chance ignore filter before running.

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
