# We own the camera: per-eye stereo override works in gameplay

**Date:** 2026-08-21 (evening) · **Status: milestone reached, verified by eye**

Tonight the per-eye camera override ran in retail Far Cry 2 gameplay for the
first time — and it works. With the override enabled, the entire world shifts
coherently: terrain, objects, everything moves together as the virtual eye
moves. Wiggle mode (the eye alternating every frame) is exactly the core of
AER (alternate-eye rendering) as seen on a flat monitor.

## The twist that made it work

The camera we "found" earlier today at constant registers c12/c16 turned out to
be the **sun shadow cascade**, not the eye camera: its projection is
orthographic, the rig hovers at exactly Z = 1000 with a pitch-free basis, and
it steps on a 64-unit texel-snapping grid while following the player. The
shadow pass writes its constants **first** each frame and the HUD writes
**last**, so both our first-write and last-write captures missed the real
camera, which writes mid-frame.

So the override hardcodes nothing. It classifies every matrix upload as it
happens: a genuine Dunia perspective transform carries a **unit-length
view-depth axis in its bottom row** (the engine is Z-up / Y-forward), which
ortho shadow blocks never have. Matching uploads get the per-eye offset
`M' = M·T(−s·right)`, with camera-right read straight out of the matrix's own
first row — no view/projection separation required, and the camera-relative
(rotation-only) variant is covered by the same math.

## What the session established

- **Gameplay perspective registers:** the main-pass camera appears at **c28**
  (free rotation + world-space translation), with further perspective passes at
  c71, c75, c87; the menu scene uses c0, c4, c8, c24.
- ~19 perspective uploads per frame get the offset; hotkeys (toggle,
  separation, eye mode, rigid-view mode) all verified live in-game.
- Separation values that looked right (~0.001–0.07) support **Dunia units ≈
  meters** — a real IPD would be ~0.065.

## Next

AER to a real headset is now the shortest path to the North Star: alternate
frames already carry alternate eyes, so what remains is submitting them to the
VR runtime — which means solving the **D3D9 → OpenXR/OpenVR texture bridge**
(D3D9Ex shared surfaces into D3D11, then runtime submission), plus feeding
headset pose into the same override math for head tracking.
