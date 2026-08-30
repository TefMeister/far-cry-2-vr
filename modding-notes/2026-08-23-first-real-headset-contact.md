# 2026-08-23 — First real-headset contact: the bridge works on a Quest 3

**The v0.1.0-alpha release was installed and tested on the home PC with a real
Quest 3 (Virtual Desktop → SteamVR), and it worked on the first try** — the
user's words: "it did, it actually did."

## Facts from the run (`%LOCALAPPDATA%\FC2VR\fc2vr.log`)

- Install = exactly the release instructions: `winmm.dll` + `openvr_api.dll`
  into `…\Far Cry 2\bin\`, SteamVR running, `bin\FarCry2.exe` launched
  directly, F10 at the menu.
- OpenVR came up against real hardware: `IVRSystem_026 / IVRCompositor_029`,
  **recommended per-eye target 2496×2688** (Quest 3 via Virtual Desktop — the
  dev-PC verification had only ever seen the null driver).
- Bridge LIVE at 1176×664 backbuffer capture; **1719 frames submitted** in the
  first session and 577 in a second toggle cycle; clean shutdowns both times;
  **zero Submit errors**.
- v0.1 expectations held: monoscopic, stretched across each eye's FOV, no head
  tracking — evaluated seated, briefly, as intended.

## Also noted (vanilla, NOT the mod)

A flat session run BEFORE the proxy ever loaded (no FC2VR log dir existed)
showed classic FC2-on-modern-hardware jank: NPCs/ragdolls bouncing on the
ground, lighting/texture flicker and pop-in, glowing foliage shadows — known
symptoms of uncapped high FPS + modern drivers on the 2008 renderer. Worth
keeping in mind when judging mod-induced vs vanilla artifacts; an FPS cap
(~60) is the usual mitigation and may also be kinder to the VR bridge's
readback path.

## What this unblocks (dev PC)

The home-machine gate is CLEARED. Next per the plan: AER stereo (sync wiggle
parity with per-eye Submit, double-buffered D3D11 textures), then a sane
virtual screen / matched projection instead of the full-FOV stretch, then head
tracking (HMD pose → the M' = M·T(t) override), later the GPU-only
shared-surface path to kill the CPU readback. User lag impression to be added
to STATUS when reported.
