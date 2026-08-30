# Phase 0 static recon — Far Cry 2 (Dunia) — 2026-08-21

First static pass over the shipped Steam binaries. No process was launched and
nothing was injected; this is pure on-disk PE + string analysis. Distilled
conclusions are folded into the engine-research dossier — this file is the raw
record and method.

## What was examined

Install: `D:\Program Files (x86)\Steam\steamapps\common\Far Cry 2`
Binaries: `bin\FarCry2.exe`, `bin\Dunia.dll`, `bin\FC2.dll`
Config: `%USERPROFILE%\Documents\My Games\Far Cry 2\GamerProfile.xml`,
`bin\FC2Init.ini`

Tools: `llvm-mingw` `objdump`, Python `pefile`, `strings`, PowerShell
`VersionInfo`.

## Key findings

1. **All three binaries are 32-bit (`pe-i386`).** Injected code must be x86.

2. **`FarCry2.exe` is a 27 KB launcher stub.** It imports only `KERNEL32`,
   `MSVCR80`, and **three** exports from `Dunia.dll`:
   - `?RunGame@@YA_NPAUHINSTANCE__@@PBD@Z` — `bool RunGame(HINSTANCE, const char*)` (the real entry)
   - `RegisterGameFunctionProvider`
   - `AddFunctionCB`
   → **All engine logic is in `Dunia.dll` (19.2 MB).** That's the RE target.

3. **`FC2.dll` is not engine code** — internal name
   `paul_dll_activate_and_play.dll` (an activation/launch shim). Skip it.

4. **Renderer: D3D9 default, D3D10 alternate.**
   - `Dunia.dll` statically imports `d3d9.dll` + `d3dx9_38.dll`.
   - Dynamically references `d3d10.dll`, `d3d10_1.dll`, `dxgi.dll` (strings) and
     exports `FCB_Benchmark_IsD3D10Supported`.
   - `GamerProfile.xml` → `<RenderProfile Platform="d3d9" …>` — D3D9 is the
     active path on this machine. DX10 quality block (`customd3d10`) unlocks
     HDR-FP32.

5. **404 named exports in `Dunia.dll`** (full dump:
   `../../..` → engine-research `dunia-exports.txt`). Dominated by the Far Cry
   **Editor** C-API (`FCE_*`) and Benchmark API (`FCB_*`). Notable for VR:
   the **`FCE_Camera_*`** family (14 fns: Get/SetPos, Get/SetAngles, Rotate,
   GetFOV, Get{Front,Right,Up}Vector, Get/SetSpeed) — editor wrappers over the
   engine camera object, a strong lead for locating the camera struct and for
   an in-process camera-drive harness. Also `FCE_Engine_UpdateViewport`.

6. **Middleware (Dunia string counts):** Havok (25), RealTree vegetation (102),
   amBX (53), EAX audio + Bink video (imported), PunkBuster (15, MP-only).
   **No** PhysX / OpenAL / FMOD. **No SecuROM** (0) — Steam build, DRM stripped;
   only a vestigial `uplay` string.

7. **Archives:** `Data_Win32\` holds Dunia FAT/DAT pairs (`common`, `patch`,
   `shadersobj` = 249 MB of compiled shaders, `sound*`). `patch.*` overlays
   base. Config is XML, mostly packed inside the archives.

8. **Version:** engine reports `0,1,0,1`; game is patch **1.03**
   (per `PatchNotes.txt`).

## Injection foothold candidates (none tried)

`eax.dll` (already a loose file in `bin\`, small, non-critical → cleanest proxy),
`d3d9.dll`, `binkw32.dll`, or `dinput8.dll` proxy; or an injector into
`FarCry2.exe`. Leaning `eax.dll` or `d3d9.dll`.

## Not done yet

No dynamic analysis, no debugger attach, no hook, no console-existence test, no
frame structure, no camera-delivery confirmation. Those are Phase 1+ (see
PLAYBOOK).
