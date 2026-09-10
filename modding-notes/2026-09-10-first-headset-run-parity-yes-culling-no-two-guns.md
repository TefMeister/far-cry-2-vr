# 2026-09-10 — first headset run: parity yes, head rotation yes, culling no, two guns

Home PC `RTX`, Quest 3 over Virtual Desktop → SteamVR, the proxy's own OpenVR bridge
submitting the captured backbuffer per eye. Tefa driving, one launch, `bin\FarCry2.exe` direct.
Build `winmm.dll` 135,168 B (`8c67162f807f`), deployed 2026-09-07.

## What the log says (`%LOCALAPPDATA%\FC2VR\fc2vr.log`)

| key | line | reading |
| --- | --- | --- |
| Num0 | `openvr_api.dll loaded from game dir` → `bridge LIVE — capturing backbuffer, submitting mono to both eyes` | bridge up |
| Num5 | `override ENABLED (mode=wiggle sep=0.0650)`; then `AER: left 559, right 559, mono 5026` | **mono stopped climbing at 5026 the moment the override went on; left == right thereafter** — parity works `[verified-live 2026-09-10, n=1]` |
| Num8 ×2 | `eye mode -> left`, `eye mode -> right`; `AER: left 1014` frozen while `right` climbed 616 → 3688 | fixed modes submit one eye only; the other holds its last frame |
| Num7 | `head tracking ENABLED (rot=1 pos=1 scale=1.00 handed=+1)`; `applied to 25644 uploads, camera-position solve FAILED on 4060` (later 6176/27018, 4508/31042) | rotation applied; ~16 % of uploads cannot recover a camera position |

## What Tefa saw

> pressing 5 brings the eyes closer together but not close enough, i can still see 2 weapons.
> 8 freezes one eye first, 2nd press freezes the other eye. 7 gives me head tracking, but the
> world scale is off and looks warped when i turn my head, also looking behind me things don't
> render, there is no black void but only ground and sky, nothing else is showing until i turn
> with my mouse, then things pop into existance

## What it means

1. **Culling.** The head rotation lives in the view-projection upload, *after* the engine has
   culled against the mouse-look camera. Terrain and sky survive (not frustum-culled the same
   way); everything else behind you is simply never submitted. This is Psychonauts' void in a
   different costume, and the answer is the same shape: the head yaw has to reach the *game's*
   camera (mouse-delta injection from the head pose), with the upload-side rotation carrying only
   the residual. Until then head tracking is a demo.
2. **Two guns.** At a real-IPD separation the world fused enough to judge and the first-person
   weapon did not. The offset goes onto every perspective viewproj at `c28`
   (`offset 5898 persp / 0 rigid`); the weapon is drawn through its own projection, so the same
   view-space translation is a far larger disparity on it. It needs its own separation.
3. **Solve failures on ~16 % of uploads** — a per-pass subset gets rotation without position.
   Plausible contributor to "warped" alongside 1. Log which passes.
4. **Handedness is right.** `Num2` was never needed; the world did not turn *with* the head.
5. **Frozen eye in fixed modes** is by design (flat-monitor test modes) and reads as a fault in a
   headset; either resubmit the held frame or say so in the help.

World scale cannot be judged yet — "the world scale is off" is not separable from 2 until the
weapon is fixed.

## Board after this

The `[VR @home]` row is answered and removed; four `[PD]` rows added (culling ⭐⭐, weapon
separation ⭐⭐, solve failures ⭐, fixed-mode note ⬇️) and a `[VR] ⬇️` re-wear after the two ⭐⭐.
