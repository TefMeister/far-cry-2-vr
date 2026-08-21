# SteamVR bridge works end-to-end; first public release (v0.1.0-alpha)

**Date:** 2026-08-21 (late evening) · **Status: bridge verified against the
compositor on the dev PC; headset test queued for the home machine**

Hours after the camera-override milestone, the second hard problem fell: the
game's frames now reach the SteamVR compositor from inside the 32-bit process.

## Why OpenVR, not OpenXR

The plan said "D3D9 → OpenXR bridge", but recon killed that cleanly: Far Cry 2
is a 32-bit process and SteamVR ships **no 32-bit OpenXR runtime** (only
`steamxr_win64.json`; the 32-bit registry key has no ActiveRuntime), so an
in-process OpenXR loader would find nothing to load. OpenVR, however, fully
supports Win32 x86 — SteamVR ships `bin/win32/openvr_api.dll` — and it is the
same path our XIII mod already proved against a real Quest 3 via Virtual
Desktop. OpenXR remains possible later through a 64-bit companion process.

## The v1 bridge

CPU readback by design (safety first, speed later): backbuffer →
`GetRenderTargetData` → system-memory surface → dynamic D3D11 texture →
`IVRCompositor::Submit` to both eyes (mono), using OpenVR's flat C API
(FnTable) for ABI safety. The game's device is untouched — no D3D9Ex promotion
yet. Everything runs on the render thread in the Present hook, initializes
lazily on the **F10** hotkey, and fails safe.

**Verified live on the dev PC** (SteamVR null driver, no headset): OpenVR init
in-process, surfaces created, frames submitted with zero compositor errors, a
788-frame run, clean F10 shutdown, and instant re-init — including during
gameplay.

Two gotchas worth remembering:

- With SteamVR running, launching the game **through the Steam library hangs
  on "Launching…"** (Desktop Game Theatre interferes). Launch `bin\FarCry2.exe`
  directly.
- The log-driven remote-observation workflow (one person plays, the assistant
  watches the log live) keeps working well for verification.

## First public release

Under the new release policy (publish continuously, with a clear
what-it-is/what-it-isn't disclaimer and a motion-sickness caution on every
release), **v0.1.0-alpha** is now public on the `far-cry-2-vr-mod` repository:
camera control + SteamVR bridge, with install instructions and third-party
licenses in the ZIP.

## Next

Home-machine headset test (Quest 3 via Virtual Desktop): expect the flat game
screen in the headset — mono, stretched, no head tracking; judging only that
the image appears and roughly how laggy it is. After that: alternate-eye
stereo (sync the wiggle parity with per-eye submission), a sane virtual-screen
projection, head tracking into the camera override, and eventually the
GPU-only capture path to remove the CPU readback.
