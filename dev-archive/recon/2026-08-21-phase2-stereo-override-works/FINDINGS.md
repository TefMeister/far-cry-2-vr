# Per-eye camera override WORKS in gameplay — 2026-08-21 (evening)

**Milestone: we own the Far Cry 2 camera on the render path, verified by eye in
retail gameplay.** With the override enabled, the whole world shifts coherently
— terrain, objects, everything moves together. Wiggle mode (alternating the eye
every frame) is, on a flat monitor, exactly the core of AER (alternate-eye
rendering). Raw log: `stereo-override-first-success.log`.

## What ran

`proxy-winmm/src/stereo.c` (staging commit `8b785d7`) inside the
`SetVertexShaderConstantF` hook. No hardcoded registers: every 4-vec4 upload is
classified at upload time — **perspective ⇔ bottom row is a unit-length
view-depth axis** (Dunia is Z-up / Y-forward), which cleanly excludes the ortho
shadow-cascade blocks and sparse swizzles. Matching uploads get
`M' = M·T(−s·right)` with `right = normalize(row0.xyz)` — only the translation
column changes, no P/V split needed.

## Confirmations from this session

- The earlier c12/c36/c16 "camera" was indeed the **sun shadow cascade** (see
  the phase-1 CAMERA-FOUND addendum). The detector correctly never flagged
  c12/c16 as perspective.
- **Perspective viewproj registers observed:** menu = c0, c4, c8, c24;
  gameplay adds **c28, c71, c75, c87**. The gameplay main-pass camera showed at
  **c28** with free rotation and world-space translation, e.g.
  `[-0.7196 -0.3881 0.0847 2666.75] / [-0.2714 0.1241 1.2263 256.91] /
  [-0.8970 0.4089 0.1686 829.12] / [-0.8969 0.4089 0.1686 829.13]` — note
  row 2 ≈ row 3 (z' ≈ w', the near-offset / infinite-far perspective pattern).
- Throughput: ~19 perspective uploads offset per frame (~17k–25k per 900-frame
  report interval); rigid-view offsetting (F9) adds ~5/frame.
- Hotkeys all verified live: F5 toggle, F6/F7 separation (÷/×1.5), F8
  wiggle/left/right, F9 rigid-view. Log-driven remote observation of an
  in-person test session worked well.
- World scale: separation values that felt reasonable sat in the ~0.001–0.07
  range, which supports Dunia units ≈ meters (IPD ≈ 0.065).

## Next

1. **AER to a headset** is now the shortest path to the North Star: submit
   alternating frames to left/right eyes (frame parity already exists in the
   wiggle mode). Blocker to solve: the **D3D9 → OpenXR/OpenVR bridge**
   (D3D9Ex shared surfaces → D3D11 texture → runtime submission).
2. Alternative: side-by-side stereo on the flat monitor first (two viewports
   per frame) — more invasive per-frame work (needs scene re-issue or draw
   interception), so AER likely wins.
3. Head tracking: feed headset pose into the same `M' = M·T(t)` (plus a
   rotation composition) — the override already proves the injection point.
