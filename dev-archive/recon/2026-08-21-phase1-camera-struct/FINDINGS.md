# Phase 1 — camera struct recovered by static disassembly — 2026-08-21

Follow-on to the Phase 0 recon. Still fully static (no debugger, nothing
injected). We disassembled the `FCE_Camera_*` exports of `Dunia.dll` with
capstone and reconstructed the engine camera object.

## Method

The map-editor C-API exports (`FCE_Camera_GetPos`, `…GetFOV`,
`…GetFrontVector`, etc.) are thin accessors that read/write the engine's live
camera. Disassembling the *getters* directly exposes the struct field offsets;
the *setters* expose the write path. Addresses below are at the preferred
ImageBase `0x10000000`.

## The camera singleton

Every accessor dereferences the same global:

```
mov eax, dword ptr [0x1164FE7C]   ; eax = current camera object
```

`0x1164FE7C` (RVA `0x0164FE7C`) holds a `CCamera*`. A byte-scan of the
executable sections found **101 references** to this global — it is a core,
widely-used object, which strongly suggests it is the real game camera and not
merely an editor construct. (Runtime confirmation still pending — see below.)

## Struct layout (offsets from the object pointer)

Reconstructed from the getter disassembly:

| offset | field | evidence |
|--------|-------|----------|
| +0x0C, +0x10, +0x14 | position x, y, z | `GetPos`: `fld [eax+0xC]`, `[eax+0x10]`, then `add eax,8; fld [eax+0xC]` |
| +0x2C | FOV | `GetFOV`: `fld [eax+0x2C]; ret` |
| +0x4C, +0x50, +0x54 | front vector | `GetFrontVector` |
| +0x58, +0x5C, +0x60 | up vector | `GetUpVector` |
| +0x64, +0x68, +0x6C | right vector | `GetRightVector` |
| +0x70, +0x74, +0x78 | euler angles | `GetAngles` |

So the object carries position, an orthonormal right/up/front basis, a FOV
scalar, and euler angles — a complete, conventional camera. For VR this is ideal:
the basis vectors are the view rotation directly, and FOV at +0x2C is the
projection knob.

## The write path (important nuance)

The setters are NOT plain field writes:

```
; FCE_Camera_SetPos (abridged)
mov  ecx, [0x1164FE7C]      ; camera
... build vec3 on stack ...
lea  eax, [ecx+0x70]        ; &orientation sub-object
call 0x10837A10             ; transform-update helper

; FCE_Camera_Rotate reads angles at +0x70/+0x74/+0x78, adds the delta,
; then also calls 0x10837A10 with [ecx+0xC].
```

Helper `0x10837A10` additionally loads another global `0x1164FE04` (probably the
scene / collision manager — matches the editor's "clip camera to terrain"
setting) and calls into it. Implication: **overriding the camera by writing the
struct directly may be ignored or clobbered**; a correct override likely has to
go through this helper, run *after* it each frame, or be applied downstream at
the GPU boundary. Not yet tested — flagged for Phase 4.

## Open question this does NOT answer

Whether `0x1164FE7C` is populated and drives rendering during real gameplay
(vs. only in the editor). That needs a runtime read. The Phase-1 winmm proxy in
`-staging` includes a read-only `camera_probe` thread that logs this exact
pointer + fields once a second, precisely to settle it.

## Follow-on: engine / viewport / console globals

Disassembling more `FCE_Engine_*` / `FCE_Core_*` exports turned up several more
useful globals (raw disassembly in `disasm-engine-viewport-console.txt`):

- **Console exists.** `FCE_Engine_IsConsoleOpen` = `return *(uint8_t*)(
  *(void**)0x11606280 + 0x69)`. So `0x11606280` (RVA `0x1606280`) is the
  console/UI manager and `+0x69` is its open flag. Good Phase-2 harness lever.
- **View/viewport object** at `0x11609560` (RVA `0x1609560`):
  `FCE_Engine_UpdateViewport` writes width→`+0x20`, height→`+0x24`, then calls
  `0x103f8ab0`.
- **`0x103f8ab0` is an observer dispatcher**, not the projection math: it walks
  listener arrays (element stride `0x14`) and calls `vtable+4` on each — a
  "viewport changed" notification fan-out. The projection rebuild is inside one
  of those callbacks; find it dynamically once hooked.
- **Angle→basis** conversion: `FCE_Core_GetAxisFromAngles` (helper `0x10050990`)
  — the rotation-convention reference for building per-eye view matrices later.
- **World/time manager** at `0x1164d594` (`GetTimeOfDay` uses `+0xc8`).

## Artifacts

- `disasm-fce-camera.txt` — raw capstone disassembly of the camera accessors.
- `disasm-engine-viewport-console.txt` — engine/viewport/console disassembly.
- Camera offsets are encoded as C macros in
  `-staging/proxy-winmm/src/camera.h`.
