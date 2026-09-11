# The head has to turn Dunia's own camera — vorpX's developer says so on the record — and the weapon needs its own separation

**From:** `/gr` (estate sweep, 2026-09-11) · **For:** the modding lane, to fold into
`ENGINE-DOSSIER.md` — the head-tracking/culling section and a new note on the first-person weapon

**Full write-up:** [`external-research/topics/2026-09-11-vorpx-says-the-matrix-cannot-fix-culling-and-the-weapon-has-a-fix-ladder.md`](../../external-research/topics/2026-09-11-vorpx-says-the-matrix-cannot-fix-culling-and-the-weapon-has-a-fix-ladder.md)

Both ⭐⭐ `[PD]` rows from the 2026-09-10 headset run have substantial public prior art. The most
useful part **closes a direction** rather than opening one.

## 1. ⭐ The culling row: there is no matrix edit that fixes this, and that is on the record

On the vorpX forum thread *Decouple Head Rotation From Mouse Movement*, a user (2020-05-08) describes
our exact symptom unprompted — forcing camera rotation independently of the game's camera means
*“by turning around you'd see either severely incomplete geometry or the void”*, because the game only
renders geometry in the original camera's view. In the same thread **Ralf, vorpX's developer, states
that decoupling head rotation from the game camera “isn't really possible to do this with the FullVR
play style”** `[reported 2026-09-11, 2020-05-04]`. vorpX maps head rotation to **mouse movement**
instead, keeping positional tracking and head roll in the matrices.

**Suggested dossier change:** record this as a closed route. The industry's most-used injector reached
our conclusion and took the same exit; nobody should spend another session hunting a downstream fix.

### The three routes and what each actually buys

- **(a) synthetic MOUSE DELTA** — vorpX's own *“mouse emulation head tracking”*, its fallback when
  DirectVR is unavailable. Cheap, guarantees culling follows. Costs, from **opentrack's** issue
  tracker (#113, #120, #803, 2014–2019) rather than from any one game: games that recentre the cursor
  each frame and read the delta **spin uncontrollably** against absolute positioning (inject
  *relative* deltas, never `SetCursorPos`); emulation **seizes the mouse**, so head-look and mouse-aim
  are by design the same axis; **jitter survives smoothing filters**; and the mapping is
  **non-linear** — 45° of head rotation reported as 50–60° in game — so the route is open-loop and
  wants a correction term.
- **(b) write the camera rotation in memory** — what **vorpX DirectVR** is (a scanner that finds the
  address holding camera rotation; Ralf calls finding it *“a fairly complex matter”* with a real chance
  of failure) and what Vireio's **VRBoost** does via per-game rule files.
- **(c) relax the culling** — **peripheral slack only.** **NVIDIA RTX Remix's Anti-Culling System** is
  the reference design (keep previous-frame out-of-frustum instances in the BVH, dedupe by hash and
  location) and **its own docs state it cannot conjure draw calls the game never submitted** and cannot
  know whether a game culls by frustum, octree or something bespoke. LukeRoss records the cruder
  version's cost: after patching FOV wider, *“the game … didn't load some parts of it in time”*.

### ⭐ The known-good architecture is HYBRID, and it matches what our own Psychonauts work already says

**R.E.A.L.** ties in-game camera **yaw to headset relative yaw** and **pitch to headset absolute
pitch**, then applies *“a supplementary view fix during rendering to perfectly correct the camera
angles and position”*. **fear-vr** (LithTech, active 2026) rotates the game camera and preserves
physical pitch/roll, then renders twice per eye. So: **coarse rotation into the game's camera, residual
and per-eye at render time** — the same split the dossier already describes as “do not rotate twice”.

⚠️ **The write must be re-asserted, not issued once.** R.E.A.L.'s README carries fixed-yaw handling
for corner cases *“where the game wrestles for camera ownership”* — direct evidence the game fights
back each frame. And ordering matters: write **after** the engine's own camera update, not on a timer
`[inferred-static 2026-09-11]`.

## 2. The weapon row: why one separation cannot serve both, and a ladder with a cheap second rung

The stereo correction the whole HelixMod/3Dmigoto ecosystem rests on is a clip-space shift
**proportional to `w`**: `clip.x += EyeSign * Separation * (clip.w − Convergence)` (bo3b's School for
Shaderhackers; 3Dmigoto's `StereoParams` = separation, convergence in `w` units, eye sign)
`[reported]`. **`w` comes from whichever projection drew that geometry**, and a viewmodel is rendered
in a separate pass with its **own FOV and a much nearer near-plane** so it does not clip into walls
(stated in Flax Engine's “render FPS weapon” docs and in Unreal/gamedev.net threads). **Hence a
separation that fuses a 20 m wall drives a 40 cm gun past the fusion limit.** That is our two guns,
and it is why lowering separation helped without resolving it.

1. **Lower global convergence** — HelixMod's Crysis 2/3 pages say it must be *“quite low”* or the
   weapon looks too close. **Flattens the world**: the wall our row already hit.
2. **⭐ Hotkeyed low-separation preset while AIMING — implementable now, on a key we already own.**
   **DHR's own 2013 Far Cry 2 3D-Vision fix binds `O` (low, for aiming) and `P` (normal)**, and
   **DarkStarSword's 2014 improvement switches convergence while the right mouse button is held.**
3. **Per-draw override** filtered by shader / index-buffer / texture hash — geo-11's notes describe a
   weapon convergence fix *“heavily filtered by index buffer and texture to not break other things”*,
   with honest residual bugs on two or three textures.
4. **Move the weapon instead of correcting it.** R.E.A.L. *“pushed weapon models away from the eye
   camera”* and *“moved the weapon so that it aligns with the dominant eye (selectable)”* — hotkey `T`
   — and made **crosshair depth dynamic**, matching the aimed object. **CryEngine 3 shipped this
   natively** (`r_StereoNearGeoScale`; near geometry pushed into the screen, zero-parallax distance
   reduced when objects are close) — presented at GDC Online 2010. ⚠️ that manual page now redirects
   and was read via search-index snippets, so it is weaker sourcing.

⚠️ **No authoritative source was found for any named VR mod simply HIDING the weapon.** Folklore, as
far as this search reached.

## 3. ⭐ Three Far Cry 2 / Dunia prior-art items that bear directly on cost

- **vorpX DirectVR reportedly WORKS in Far Cry 2** — a user (2020-07-29) reports it functioning but
  only from *“bed saves”*, not menu saves, **and that weapons are “not in scale with the rest of game
  elements”**, plus black bands `[reported]`. **Two consequences:** route (b) is **not speculative on
  this engine** — something already finds a Dunia camera-rotation address; and **our weapon defect is
  an already-known vorpX symptom in this exact game**, not a novel finding.
- **HelixMod's Far Cry 2 (DX9) 3D-Vision fix** — DHR, **2013-01-04**: fixes the crosshair (rest of HUD
  left at screen depth) and effects (smoke, water, dust, fire). Unmaintained, still up. ⚠️ **It does
  not claim to fix the weapon model**, consistent with rungs 1–2 being enough under 3D Vision.
- **⚠️ Far Cry 2 Multi Fixer** (FoxAhead) injects `FarCry2MF.dll` and **patches Dunia in process
  memory at runtime rather than editing files, explicitly to survive Steam integrity checks**, for FC2
  1.03. **Worth reading before we patch anything — it is both a precedent and a collision check.**
- **Cheap static check this pass suggests:** community reports place an `fFOV` desired-FOV multiplier
  in **`25_cameras.xml`** in Dunia game data `[reported, unverified]`. We can read the archives; one
  look confirms or kills it.

⚠️ **No public Far Cry 2 VR mod exists** beyond the vorpX profile, and **no published Dunia camera
yaw/pitch addresses or view-matrix offsets** could be found — so route (b) has an existence proof and
no published coordinates. The open-source **Far Cry 1 VR mod** (fholger) is **not transferable**: it
builds against the CryEngine Mod SDK with engine-level access.

## 4. Cross-project note

Nothing public was found on **Unreal Engine 2** camera or culling in a VR-injection context — UEVR's
cvar approach is UE4/5 only. That bounds what the sibling `psychonauts-vr` and `XIII2003-vr` projects
should expect from searching, and is recorded here rather than filed separately to avoid duplicate
drops.
