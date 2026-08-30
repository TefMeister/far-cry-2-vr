# Phase 1 — first live debug session — 2026-08-21

First time running against the live game. Built the winmm proxy, ran it in-game,
then attached x32dbg and read memory directly.

## The foothold works

The 32-bit winmm proxy loads into `FarCry2.exe`, forwards all 180 winmm exports
(game boots fine), logs to `%LOCALAPPDATA%\FC2VR\fc2vr.log`, and its read-only
camera probe runs. Confirmed across three launches: banner logged, `Dunia.dll`
base `0x10000000`, camera-global resolved.

## The key result: FCE editor globals are null in retail gameplay

The winmm probe reported the editor camera pointer `*(0x1164FE7C)` as **NULL** at
the menu AND after loading into gameplay. Attaching x32dbg to the live in-game
process and reading memory confirmed it directly:

| global | meaning | in-game value |
|---|---|---|
| `0x1164FE7C` | editor camera ptr | `0x00000000` (null) |
| `0x11606280` | console/UI mgr ptr | `0x00000F18` (not a valid ptr) |
| `0x11609560` | view/render obj ptr | `0x10F17102` → points at a **string constant** `"::getData(unsigned char*,...)"`, i.e. stale/uninitialised |
| `0x1164FE04` | scene/collision mgr | `0x00000000` |
| `0x1164D594` | world/time mgr | `0x00000000` |

**Conclusion: the entire `FCE_*` (Far Cry Editor) API is editor-only.** Its
globals are only populated when the map editor runs. Great for static layout
recovery; useless as live gameplay hooks.

## The game camera is a different class

Captured a full in-game memory snapshot (720 MB, 1360 regions + registers) with
the state-snapshot skill and scanned it offline for the editor camera's exact
signature — three orthonormal basis vectors at +0x4C/+0x58/+0x64, FOV at +0x2C,
finite world position at +0x0C. Across 197 private-RW regions this found only 4
candidates, all false positives (positions ~0.97, near axis-aligned). **So the
retail camera does not share the editor camera's struct layout.** The recovered
layout is valid for the *editor* camera only.

## Workflow lessons (recorded so we don't repeat them)

- **Do not launch FC2 under the debugger and detach** — it leaves DirectInput
  broken (mouse dead in-game). Launch normally, then *attach* to the running pid.
- **Disable event breakpoints first.** Dunia names threads via
  `MS_VC_EXCEPTION (0x406D1388)`; with default first-chance breaking, x32dbg
  stops on every thread. Turn off `[Events]` DllEntry/TlsCallbacks/DllLoad/
  EntryBreakpoint/ThreadEntry (and/or add a first-chance ignore filter).
- Attaching to the already-running in-game process, pausing, reading, and
  detaching worked cleanly.

## Next

Get the camera from the **D3D9 render path**: hook `Present`/`EndScene` (dummy-
device vtable grab; MinHook already vendored in the proxy), capture the real
device, then find the view/projection matrix via `SetTransform` or
`SetVertexShaderConstantF`. That is the reliable, general path and it sidesteps
the editor-API dead end entirely.
