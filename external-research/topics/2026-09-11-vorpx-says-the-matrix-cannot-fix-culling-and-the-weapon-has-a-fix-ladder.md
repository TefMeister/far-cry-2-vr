# Both ⭐⭐ rows have public prior art: vorpX's developer says the matrix alone CANNOT fix culling, and the weapon has a four-rung fix ladder

**Researched:** 2026-09-11 (`/gr`, estate sweep) · **Project:** `far-cry-2-vr` · **Engine:** Dunia (Ubisoft Montreal), D3D9, 32-bit

## Why this matters here, specifically

The 2026-09-10 first headset run produced two ⭐⭐ `[PD]` rows, and both turn out to be well-trodden
ground in public sources — including, in one case, **a statement from the developer of the most-used
VR injector in existence that our current approach cannot work**, and in the other, **a ladder of four
escalating fixes the stereo-3D community settled on years ago**. There is also a prior-art data point
specifically on Far Cry 2 that bears directly on how expensive the first fix will be.

---

## 1. The culling row: the defect is documented verbatim, and the matrix alone cannot cure it

Our row says the head turns the picture but not the engine's camera, so nothing behind the player is
drawn. On the **vorpX forum thread "Decouple Head Rotation From Mouse Movement"**, user **Eola667
(2020-05-08)** describes the identical symptom unprompted: forcing camera rotation independently of
the game's camera means *"by turning around you'd see either severely incomplete geometry or the
void"*, because the game only renders geometry in the original camera's view `[reported 2026-09-11]`.

And in the same thread, **Ralf — vorpX's developer — states plainly that decoupling head rotation
from the game camera "isn't really possible to do this with the FullVR play style"**
`[reported 2026-09-11, 2020-05-04]`. vorpX instead **maps head rotation to mouse movement**, handling
positional tracking and head roll separately in the matrices. A third user in the thread adds that
vorpX's FOV widening already runs into culling limits in games like Mass Effect.

**⭐ This is the most useful thing in this topic, because it closes a direction rather than opening
one.** The industry's most-used injector reached our conclusion and took the same exit. There is no
clever matrix edit waiting to be found. Recorded plainly so nobody spends another session looking.

### The three routes, what each actually buys, and who uses it

**(a) Head yaw/pitch as synthetic MOUSE DELTA.** vorpX's own name for it is *"mouse emulation head
tracking"*, with a head-tracking-sensitivity control; Ralf describes it as the fallback when DirectVR
(below) is unavailable `[reported, 2024-03-30]`. Community reporting says Vireio Perception does roll
in the view matrix and yaw/pitch by mouse emulation `[reported, community, not developer-authoritative]`.
The documented costs are specific and come from **opentrack's own issue tracker** (issues #113, #120,
#803, 2014–2019) rather than from any one game, which is what makes them worth trusting:

- **Games that recentre the cursor every frame and read the delta fight absolute-position emulation**
  and spin uncontrollably. Inject *relative* deltas (raw input, or a hooked `GetDeviceState`) — never
  `SetCursorPos`.
- **Emulation seizes the mouse**, so with this route head-look and mouse-aim are *by definition the
  same axis*. Aim decoupling is not a refinement you add later; it is excluded by the design.
- **Micro-jumps and stutter survive acceleration and smoothing filters** (#803).
- **The mapping is non-linear** — 45° of real head rotation reported as 50–60° in game, because the
  game's own sensitivity and acceleration curves sit between you and the camera. Recentring drift
  follows from that, so the route is open-loop and wants a correction term.

**(b) Writing rotation into the game's own camera memory.** This is what **vorpX DirectVR** is: a
memory scanner that locates the address holding camera rotation and writes it, for *"perfect 1:1"*
low-latency tracking. Ralf calls finding that single address *"a fairly complex matter"* with a real
chance of failure, which is why vorpX warns at every scan; at least one DirectVR feature exists for
150+ games `[reported]`. **Vireio's VRBoost** does the same thing driven by per-game rule files
(`cfg\VRboost_rules\*.mtbs`), though VRBoost itself is closed-source and the repo's own docs call it
outdated.

**(c) Widening or relaxing the culling.** Worth knowing precisely because of what it *cannot* do.
**UEVR** exposes `VR_DisableHZBOcclusion` and `VR_DisableInstanceCulling` for UE4/5. The most serious
public engineering of the idea is **NVIDIA RTX Remix's (dxvk-remix) Anti-Culling System**, built for
the same root cause — path tracing needs geometry the game culled — which keeps previous-frame
out-of-frustum instances alive in the BVH, de-duplicating by hash and location
(`rtx.antiCulling.object.enable`, `…fovScale`, `…light.enable`, and others). **Its documented limits
are the point:** it cannot conjure draw calls the game never submitted (so it is blind after a load),
and it cannot know whether a game culls by frustum, octree, or something bespoke. And LukeRoss
records the cost of the cruder version: after patching FOV wider, *"the game was expecting to draw a
narrower portion of the world, so it didn't load some parts of it in time"* — hard culling traded for
streaming and LOD pop-in, plus draw-call and fill cost.

**⇒ (c) buys peripheral slack only. It cannot show what is behind you. Only (a) or (b) can.**

### ⭐ The shape both successful projects use is HYBRID, not either/or

**LukeRoss's R.E.A.L.** (GTA V; README last updated 2022-07-08, development discontinued) **ties
in-game camera yaw to headset relative yaw and in-game camera pitch to headset absolute pitch**, then
applies *"a supplementary view fix during rendering to perfectly correct the camera angles and
position"* via 3Dmigoto `[reported]`. **fear-vr** (DR-89, an actively-developed OpenXR mod for F.E.A.R.
on LithTech) states HMD rotation rotates the *game camera* relative to a neutral horizontal direction
while physical pitch and roll are preserved, and the engine then renders twice with per-eye matrices
`[reported, single-author README]`.

So: **coarse rotation into the game's camera so culling and streaming follow, then the residual
correction and the per-eye offset at render time** — which is exactly the split our own dossier
already describes for Psychonauts under "do not rotate twice". ⚠️ **Ordering matters: write after the
engine's own camera update, not on a timer** `[inferred-static 2026-09-11]`.

⚠️ And a caution R.E.A.L.'s README supplies for free: it carries **fixed-yaw handling for corner cases
*"where the game wrestles for camera ownership"***. Direct evidence that the game fights back each
frame and that a write needs to be re-asserted, not issued once.

---

## 2. The weapon row: why one separation cannot serve both, and the four-rung ladder

The canonical stereo correction the whole 3D Vision / HelixMod / 3Dmigoto ecosystem is built on is a
per-vertex clip-space shift proportional to `w`:
`clip.x += EyeSign * Separation * (clip.w − Convergence)` (bo3b's School for Shaderhackers; 3Dmigoto's
wiki documents `StereoParams` as x = separation, y = convergence in `w` units, z = eye sign)
`[reported]`. **`w` comes from whichever projection drew that geometry.** Viewmodels are rendered in a
separate pass with their own projection — their own FOV and a much nearer near-plane, so the weapon
does not clip into walls (stated in Flax Engine's own "render FPS weapon" docs, and in Unreal and
gamedev.net threads on viewmodel FOV and Z-clipping). **So a separation that fuses a 20-metre wall
drives a 40-centimetre gun past the eyes' fusion limit.** That is our "two guns", and it explains why
lowering separation helped without resolving it.

**The ladder, cheapest first:**

1. **Lower global convergence until the weapon fuses.** HelixMod's Crysis 2/3 pages (2015) state
   convergence must be set *"quite low"* or the weapon looks too close. **Cost: the whole world
   flattens** — this is precisely the wall our row has already hit.
2. **⭐ Hotkeyed presets, switched while AIMING.** This is the community's standard answer and it is
   cheap for us. **DHR's own Far Cry 2 fix binds `O` (low convergence for aiming) and `P` (normal)**,
   and **DarkStarSword's later improvement (fix comments, 2014-09) switches convergence while the
   right mouse button is held** `[reported]`. Several HelixMod fixes use Mouse-2 or `H` for a
   "good aiming from the gun" low-separation preset. We already have live numpad control of
   separation, so this rung is nearly free.
3. **Per-draw / per-shader separation and convergence override**, with the weapon identified by
   **shader hash, index-buffer hash and texture hash**. geo-11's release notes describe a weapon
   convergence fix *"heavily filtered by index buffer and texture to not break other things"* — with
   honest residual bugs on two or three textures, which is a useful calibration of how clean this
   gets. This is the correct fix and what 3Dmigoto and geo-11 exist to make possible.
4. **⭐ Treat the viewmodel as geometry to MOVE, not stereo to correct — which is what the VR mods
   actually do.** R.E.A.L. states it *"pushed weapon models away from the eye camera"* and *"moved the
   weapon so that it aligns with the dominant eye (selectable)"* — hotkey `T`, right eye by default —
   explicitly so you can aim down sights; it also makes **crosshair depth dynamic**, matching the
   depth of the aimed object. fear-vr hides the upper arms, keeps hands and weapon, and attaches the
   weapon to the controller rather than the head.

**Engine-native precedent that legitimises rung 4:** **CryEngine 3 shipped a scale parameter for near
geometry (the weapon) that pushes it into the screen for stereo, and reduces the zero-parallax
distance when objects are close** so the effect fades rather than producing a depth conflict
(CryEngine 3 manual, "Tips, Tricks and Experiences using Stereo-3d"; the cvar `r_StereoNearGeoScale`
exists in CryEngine's cvar docs; presented at **GDC Online 2010** by **Carl Jones and Sean Patrick
Tracy**). ⚠️ The manual page now redirects and was read via search-index snippets rather than directly,
so treat that one as slightly weaker sourcing `[reported, weaker provenance]`.

⚠️ **A negative worth recording:** no authoritative statement was found that any named VR mod simply
*hides* the weapon. That is folklore as far as this search reached.

---

## 3. ⭐ Far Cry 2 and Dunia specifically — three concrete prior-art items

- **vorpX DirectVR reportedly WORKS in Far Cry 2.** User **cercata (2020-07-29)** on the vorpX Far Cry
  2 thread reports DirectVR functioning, but only when loading *"bed saves"* rather than menu saves;
  and that **weapons are *"not in scale with the rest of game elements"***, plus black bands
  `[reported]`. **Two things follow, and both are load-bearing for us.** First, it implies vorpX's
  scanner does find a **Dunia camera-rotation address**, so route (b) is not speculative on this
  engine. Second, **the weapon depth/scale problem in FC2 is already a known vorpX symptom** — we have
  not found a novel defect, we have rediscovered a documented one. No statement from Ralf on FC2.
- **HelixMod's 3D Vision fix for Far Cry 2 (DX9)** — author **DHR**, published **2013-01-04**. It
  fixes the crosshair (the rest of the HUD is left at screen depth) and effects (smoke, water, dust,
  fire), via `DX9Settings.ini` plus a borrowed NVIDIA Inspector profile and a ProxyLib chain.
  Unmaintained; page and files still up. ⚠️ **Notably it does NOT claim to fix the weapon model** —
  consistent with rung 1/2 having been good enough under 3D Vision, and consistent with our finding
  that the weapon is a separate problem from the world.
- **Far Cry 2 Multi Fixer** — **FoxAhead**, GitHub. A launcher that injects `FarCry2MF.dll` and
  **patches Dunia in process memory at runtime rather than editing files, explicitly to survive
  Steam's integrity checks**, for FC2 1.03 Steam/GOG/Uplay; apparently active. Useful twice over: as
  a working public precedent for runtime Dunia patching, **and as a collision check — we should know
  what it already patches before we patch the same bytes.**
- **Dunia FOV is partly data-driven:** community reports place an `fFOV` float (a desired-FOV
  multiplier) in `25_cameras.xml` in Dunia game data `[reported, unverified]` — not verified against
  FC2's own archives by this pass, and worth one cheap static check since we can read the archives.

⚠️ **No public Far Cry 2 VR mod exists** other than the vorpX profile. And the open-source
**Far Cry 1 VR mod** (fholger / Holger Frydrych — SteamVR, full 6DOF, motion controls, 160+ stars) is
**not transferable**: it is built against the CryEngine Mod SDK with engine-level access we do not
have. Its README does contribute one transferable note — distant LOD can cause stereo artifacting at
certain viewing angles.

## 4. What this unlocks

- **The culling row stops being open-ended.** Two routes, not three; (c) is a palliative; the hybrid
  shape is the known-good architecture; and the write must be re-asserted because the game contests
  ownership. The next question is narrow and static: **where does Dunia keep the camera rotation the
  engine culls against** — which our dossier's camera-matrix work is already pointed at.
- **The weapon row has a cheap first move.** Rung 2 — a separate, much lower separation while aiming,
  on a key we already own — is implementable now and is what the one public FC2 stereo fix did.
- **One collision check and one cheap static read** were added by this pass: what Multi Fixer patches,
  and whether `25_cameras.xml` is real in our archives.

## Sources

All read online; no code copied. Full credit list in `CREDITS.md`.

- **Ralf (vorpX)**, **Eola667**, **CowPox** — vorpX forum, *Decouple Head Rotation From Mouse
  Movement*, 2020-05-04 → 2020-05-08; *Head Tracking*, 2024-03-30 → 2025-03-12; *Technical Overview?*,
  2018-02 onward; vorpX features page (Geometry 3D vs Z-Buffer 3D, DirectVR, FullVR vs Immersive
  Screen).
- **cercata** — vorpX forum, *Far Cry 2*, 2020-07-29 (DirectVR works from bed saves; weapons not in
  scale; black bands).
- **DHR** — HelixMod, *Far Cry 2 (DX9)* 3D Vision fix, 2013-01-04, with **DarkStarSword**'s
  right-mouse-button convergence improvement in the comments, 2014-09.
- **davegl1234** and the HelixMod community — geo-11 announcement (2022-06-21, updated through 2026)
  and release notes (weapon convergence filtered by index buffer and texture); binaries mirrored by
  **ThreeDeeJay**.
- **bo3b** — School for Shaderhackers, *Canonical Stereo Code* (the `w`-proportional formula); **bo3b**
  and **DarkStarSword** — 3Dmigoto wiki (`StereoParams`, auto-convergence, auto-crosshair) and
  DarkStarSword's `3d-fixes` (per-game fixes including hold-aim convergence presets). ⚠️ the Canonical
  Stereo Code page 404'd at the indexed URL and was read via snippets.
- **LukeRoss** — R.E.A.L. mod README (`gta5-real-mod`), last updated 2022-07-08, discontinued; and the
  Patreon *AER v2* post on alternate-eye rendering.
- **praydog** — *UEVR: An Exploration of Advanced Game Hacking Techniques*, 2023-07-03, and UEVR's
  documentation; **vrpupu** (third party) for the UEVR advanced-settings cvar listing, 2026-01.
- **NVIDIA GameWorks** — `dxvk-remix`, *Anti-Culling System* documentation.
- **cybereality** and contributors — Vireio Perception (LGPL-3.0); VRBoost's memory-write approach,
  with the repo's own note that VRBoost is closed-source and outdated. MTBS3D forum for the
  roll-vs-yaw/pitch split `[community claim]`.
- **opentrack** contributors — issues #113, #120, #803 (2014–2019) for the mouse-emulation failure
  modes.
- **DR-89** — `fear-vr` (OpenXR mod for F.E.A.R., active 2026). **BerZerker96** —
  *6DOF Head-Tracking Mods Hub* and its "Loop" framework (40+ games) — ⚠️ mechanism **entirely
  undocumented** publicly, recorded as a gap.
- **FoxAhead** — *Far Cry 2 Multi Fixer* (runtime Dunia memory patching) and its Steam guide, 2019.
- **fholger (Holger Frydrych)** — Far Cry 1 VR mod (CryEngine Mod SDK; not transferable, one LOD note).
- **Crytek** — CryEngine 3 manual, *Tips, Tricks and Experiences using Stereo-3d*, and
  `r_StereoNearGeoScale`; **Carl Jones** and **Sean Patrick Tracy** — GDC Online 2010, *CryENGINE 3:
  Real-time Stereo 3D at Minimal Performance Cost*. ⚠️ manual page redirects; read via snippets.
- **Flax Engine** docs, *HOWTO: Render FPS weapon*; Unreal and gamedev.net community threads on
  viewmodel FOV and Z-clipping.

### Gaps this pass did not close

- **How R.E.A.L. actually sets the in-game camera** — script hook, direct memory write or input. The
  README states the effect and never the mechanism.
- **The "Loop" framework behind 40+ public 6DOF mods is undocumented.**
- **No published Far Cry 2 / Dunia camera yaw-pitch addresses, view-matrix offsets or free-cam notes.**
  FC2 cheat tables exist but none indexed documents camera angles; vorpX's DirectVR internals are
  closed by design. **So route (b) has a known-possible existence proof and no published coordinates.**
- **No quantified latency comparison** between mouse-emulated tracking and direct camera writes.
- **No record of a Far Cry 2 weapon-doubling fix specifically.**
- **Nothing found on Unreal Engine 2 camera or culling in a VR-injection context** — relevant to the
  sibling Psychonauts and XIII projects, and recorded here because it bounds what to expect there.
