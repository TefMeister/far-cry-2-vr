# Game camera located on the render path — 2026-08-21

Follow-on to the D3D9 Present hook. We hooked `IDirect3DDevice9::SetVertexShaderConstantF`
(vtable index 94, read off the same dummy-device vtable) and dumped the low
vertex-shader constant registers to find the camera. Raw dump:
`d3d9-vs-constants-gameplay-camera-found.log`.

## The trap, and the fix

First dumps only ever showed **menu/HUD** matrices (identity-ish / ortho), even
in gameplay. Reason: the HUD is drawn last each frame and overwrites the low
registers just before `Present`, so a **last-write** snapshot only ever sees UI.
Fix: capture the **first write to each register per frame** (reset the flags in
the Present hook), and widen coverage to c0..c95 (matrices go up to c71+).

## The camera

| register | matrix | notes |
|---|---|---|
| **c12–c15** | VIEW (world→view) | orthonormal 3×3 rotation + translation; `m[3]=(0,0,0,1)` |
| **c36–c39** | inverse VIEW (view→world) | **exactly `inverse(c12)`** (Rᵀ, `−RᵀT`); translation = camera world pos |
| **c16–c19** | projection | split with a **1/32 ↔ 32** scale vs the view (c0/c8 carry ~0.031 = 1/32) |

**Verification (movement correlation):** with the player walking (keyboard — the
mouse was broken), the c36 translation tracked the player: X moved 2144→2272, Y
2720→2848, Z steady ≈1000 (eye height). A static object transform would not move
with the player, so c12/c36 is the camera. Standing example: pos ≈
(2144, 2720, 1000), rotation a clean 90°-about-Z (facing an axis).

## Engine notes

- Far Cry 2 uploads **combined / scaled** matrices, not a standalone clean D3D
  perspective projection. The 1/32↔32 split is a float-precision trick for the
  huge world. This is why a naive "sparse projection matrix" heuristic fails — it
  false-flagged a trivial swizzle (`[0,0,0,0][0,0,0,0][0,0,0,1][0,0,1,0]`) at c71.
- Matrix-block upload registers seen: c0, c4, c8, c12, c16, c20, c24, c28, c32,
  c36, c71, and later c75, c83, c87, c101 (count=20), c121 (count=5).
- Caveat: the first-write value steps on a ~64-unit grid — possibly a
  sector-relative view rather than the exact main-pass camera. Cross-check
  against `shadersobj.dat`, or capture the main-pass write specifically, before
  building the per-eye override.

## Next

Per-eye override: rewrite the c12 view (and c16 scaled projection) per eye inside
the `SetVertexShaderConstantF` hook, honouring the 1/32↔32 split. Then stereo on
a flat monitor.
