# Head tracking (roadmap item 3): OpenVR's exact pose-matrix layout, the Y-up→Z-up conversion problem, and a documented alternative that sidesteps it entirely

**Status:** 🆕 new — technical reference plus a genuinely different second option, not just supporting
detail for the one approach already implied by the plan.

Follow-up to the previous topic in this repo (AER/readback/shared-surface/matched-FOV), which
deliberately skipped roadmap item 3 — "head tracking (HMD pose → the `M' = M·T(t)` override, rotation
composition on top)" — as mostly project-specific math. This pass looked specifically at that item:
how to read OpenVR's pose data correctly, the coordinate-system mismatch it creates against Dunia's
convention, and whether any prior art solves the general problem differently.

## 1. OpenVR's `HmdMatrix34_t`: exact layout, and a real gotcha in how to read it

The **official OpenVR wiki page "Matrix Usage Example"**
(`github.com/ValveSoftware/openvr/wiki/Matrix-Usage-Example`) is the canonical reference (Valve's own,
not a third party) for `GetDeviceToAbsoluteTrackingPose`'s output matrix, and worth using directly
rather than re-deriving from scratch:

- **Storage vs. math convention:** the matrix is **stored row-major in memory** (`m[0][0], m[0][1],
  m[0][2], m[0][3], m[1][0], ...`) but is meant to be **read as a column-major matrix** — i.e., the
  three rotation basis vectors are columns 0–2, and the translation is column 3, when you look at it
  as `AXx AYx AZx Tx / AXy AYy AZy Ty / AXz AYz AZz Tz`. Position is at `m[row][3]` for each row
  directly (no transpose needed just to read XYZ position). Extracting a usable 4×4 for typical
  column-vector graphics math requires an explicit **transpose** of the 3×3 rotation part — Valve's
  own `hellovr` sample code does this transpose, and it's easy to get backwards (silently produces a
  transposed/wrong rotation, not a crash) if copied without noticing the layout note.
- **Direction convention:** OpenVR describes this as a "**pose of B relative to A**" transform, which
  Valve explicitly calls out as different from the more common "transform A into B" mental model — a
  second, independent way to get a sign/direction error that looks plausible but is wrong.
- **Axis convention (confirmed elsewhere, standard OpenVR knowledge):** right-handed, **Y-up**, camera
  looks down **−Z**. (Contrast: Unity is left-handed +Z-forward, Unreal is +X-forward — OpenVR matches
  neither, so don't assume a convention from having used either engine's XR integration before.)

## 2. The coordinate mismatch this creates for Dunia, and a way to avoid hardcoding the fix

Dunia is **Z-up, Y-forward** (already established by this project's own recon —
`2026-08-21-stereo-override-works-we-own-the-camera.md`: "the engine is Z-up / Y-forward," and the
override already reads camera-right straight out of the matrix's own first row rather than assuming a
fixed layout). OpenVR is Y-up, −Z-forward. Composing an OpenVR rotation delta into a Dunia matrix
therefore needs an axis remap, not just a straight matrix multiply — general principle confirmed
across multiple sources checked (OpenVR issue discussions on coordinate systems; general graphics
references): going from a Y-up to a Z-up convention is "one view rotation away" — conceptually a fixed
90°-class permutation of which input axis feeds which output axis (OpenVR's up (Y) → Dunia's up (Z);
OpenVR's forward (−Z) → Dunia's forward (Y); OpenVR's right (X) → Dunia's right (X), signs to be
verified empirically), but **no source found gives a pre-built, verified matrix for this exact
pairing** — every real implementation seems to hand-derive and empirically verify it against their
specific target engine, not copy a universal constant.

**The concrete recommendation this points to, specific to how this project already works:** rather
than hardcoding a fixed OpenVR→Dunia axis-permutation table (easy to get a sign wrong and hard to
notice — see the transpose/direction gotchas above), **extend the same self-deriving approach the
project already uses for eye separation**. The override already reads camera-right directly from the
game's own per-frame perspective matrix's first row instead of assuming a layout, and the M' derivation
already established that the view-depth axis is the matrix's own bottom row. That means the game's own
full right/up/forward basis is already recoverable from the same matrix data being parsed every frame
— so the OpenVR rotation delta can be transformed into *that* basis directly (basis-to-basis, derived
from ground truth every frame) instead of trusting a hand-guessed fixed permutation matrix that has to
be right on the first try in a live headset test to even know if it's wrong.

## 3. A documented alternative that sidesteps coordinate conversion entirely: Vireio Perception's VRBoost

Worth knowing about even if not adopted: **Vireio Perception** (open-source D3D9/D3D11 VR injection
driver, `github.com/cybereality/Perception`) solved generic head-tracking-in-arbitrary-games with a
fundamentally different technique than matrix composition. Its **VRBoost** component "adds
head-tracking directly to game memory registers" by **masquerading tracked rotation as mouse-look
input** into whatever internal variables the game's own camera system already reads for
yaw/pitch — i.e., it doesn't touch the render-side matrix math or worry about coordinate conventions
at all. The game's own camera code does the rotation composition, in its own native coordinate system,
exactly as it already does for a mouse — because as far as the game can tell, it *is* mouse input.

- **Trade-off:** this only gives rotation (3DOF head-look), not true positional 6DOF, and it's input
  simulation rather than a frame-exact render override, so it inherits whatever smoothing/acceleration/
  dead-zone logic the game's own mouse-look code has (could feel less crisp than a direct matrix
  override).
- **Why it's worth keeping in mind:** it completely sidesteps everything in section 2 above — no axis
  remap to get right, no risk of a subtle sign error being invisible until tested in a headset. If the
  basis-derivation approach in section 2 turns out to be fiddly to verify live, this is a
  proven, documented fallback for getting *some* head rotation working quickly, and the two aren't
  mutually exclusive — VRBoost-style input injection for rotation now, full matrix-composed 6DOF
  (including position, which mouse-input can't express at all) as the real target later.
- No FC2-specific register offsets are published anywhere found this pass (Vireio never targeted this
  game) — this is a technique lead, not a ready-made memory address.

## Sources (see [CREDITS.md](../CREDITS.md) for the full standing credit)

- ValveSoftware/openvr GitHub wiki — "Matrix Usage Example" (official `HmdMatrix34_t` layout and `hellovr` sample reference).
- cybereality and the Vireio Perception contributors — `Perception` GitHub repo and project documentation (VRBoost head-tracking-via-input-masquerade technique).
- General OpenVR coordinate-system discussion (ValveSoftware/openvr issue threads) — right-handed/Y-up/−Z-forward convention confirmation.
