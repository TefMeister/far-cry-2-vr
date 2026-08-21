# 2026-08-21 — Found the game camera on the render path

Big milestone today: we located Far Cry 2's actual gameplay camera. Not through
the editor API (that was a dead end — all null in the real game), but by watching
what the game hands to the graphics card each frame.

## The path that worked

1. **Hooked Direct3D 9 `Present`** — got our code running at the frame boundary
   on the game's real device (1280×720, hardware vertex processing → shaders).
2. **Hooked `SetVertexShaderConstantF`** — the call the game uses to upload
   matrices to its vertex shaders. We snapshot the low constant registers each
   frame and dump them as candidate 4×4 matrices.
3. First try only ever showed the **menu/HUD** transforms, because the HUD is
   drawn last and overwrites the low registers right before the frame is
   presented. Fix: capture the **first** write to each register per frame (the
   3D camera is set early, the HUD late), and widen the range.

## What we found

Two registers hold the camera, and they're **exact inverses of each other** —
the classic fingerprint of a view / inverse-view pair:

- **c12** = the view matrix (world → camera space)
- **c36** = its inverse (camera → world), whose translation is the **camera's
  world position**

I verified the inverse relationship numerically, and — the clincher — when the
player **walked forward, that position moved** (X and Y tracked the movement, Z
stayed at eye height). So it's unmistakably the camera, not some coincidental
object transform. Mouse still being broken didn't matter here; walking with the
keyboard was enough to prove it.

The projection sits at **c16**, but Far Cry 2 splits it with a 1/32-vs-32 scale
factor against the view — a precision trick because its world is enormous. That
split is exactly why my quick "does this look like a projection?" auto-detector
was fooled (it flagged a trivial swizzle matrix instead). Lesson noted: this
engine uses *combined, scaled* matrices, not a tidy standalone projection.

## Why this matters

This is the keystone for VR: we can now **read the live camera every single
frame** straight off the render path. The next step is the per-eye override —
using the c12 view and the c16 scaled projection to render the world from two
eye viewpoints. Still a lot ahead (stereo, then a headset), but the hardest
"where is the camera?" question is answered.

## Loose end: the mouse

In-game mouselook has been dead for a while. It traces back to an earlier
session where I launched the game *under* a debugger and then detached — that
wedged Windows' mouse input state, and it's stuck across relaunches. It's not the
mod (the mod worked fine before that), and a reboot/sign-out clears it. Rebooting
now.

Full technical detail is in the engine dossier; the raw constant dump is saved in
the dev-archive live-debug folder; the code is in the staging repo.
