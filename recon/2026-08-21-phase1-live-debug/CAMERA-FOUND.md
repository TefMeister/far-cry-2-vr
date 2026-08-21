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

---

## ADDENDUM (2026-08-21, later): the caveat was right — c12/c16 is the SHADOW rig

Re-analysis of this same dump before writing the per-eye override shows the
gameplay c12/c36/c16 first-writes are almost certainly the **sun shadow cascade
camera**, not the eye camera:

- The gameplay c16 block is **orthographic**: its bottom row is `(0,0,0,1)`, so
  `w' = 1` — no perspective divide. Sun shadow maps are drawn with exactly such
  ortho projections; the 32 / 0.031 diagonals are cascade extents, not a
  precision split.
- The c36 rig sits at **Z = 1000 exactly**, pitch-free, with a clean 90°
  axis-aligned basis, and steps on a **64-unit grid** — classic shadow-cascade
  **texel snapping** while following the player from above. c44 carries the sun
  direction `(0,0,-1)` (straight down) and both positions: the rig at
  `(x, y, 1000)` and what looks like the **real eye at `(x, y, 35.3)`**.
- Why we never saw the real camera: the **shadow pass draws first** each frame
  (first-write capture sees it) and the **HUD draws last** (last-write capture
  sees that); the main pass writes in between.

What a real Dunia perspective transform looks like (from the menu frames, where
the 3D background has no shadow pass): the bottom row is the **view forward
axis** — `w'` is taken from view depth, e.g. c8 rows
`[0,0,-1,-0.25] / [0,0,-1,0]`. Dunia view space is Z-up / Y-forward
(CryEngine heritage), so in gameplay the depth row will look like `(0,±1,0,·)`
rotated by yaw.

**Consequence for the override:** do not hardcode c12/c16. The stereo module in
`-staging/proxy-winmm/src/stereo.c` instead classifies every 4-vec4 upload at
upload time (perspective ⇔ unit-length depth axis in the bottom row, dense upper
rows) and offsets those, logging which registers the main pass really uses. One
gameplay session will both identify the true camera registers and test the
per-eye offset.
