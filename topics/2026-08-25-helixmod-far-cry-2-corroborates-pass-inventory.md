# A real HelixMod 3D Vision fix exists for this exact game — corroborates the shadow-cascade finding and flags water/fire as separate trouble passes

**Status:** 🆕 new · **Priority:** high — corroborating third-party evidence for work already deep
in progress, plus new pass-inventory information not yet captured anywhere in this project's own
findings.

## What was found

**[Helix Mod: Far Cry 2 (DX9)](https://helixmod.blogspot.com/2013/01/far-cry-2-dx9.html)** is a real,
published NVIDIA 3D Vision stereoscopic fix for this exact game and renderer — independent
third-party confirmation that a per-eye camera/projection override is achievable here, arriving
after this project had already independently reverse-engineered its own working per-eye override
live (2026-08-21, verified "the whole world shifts, everything moves together"). This is corroborating
evidence, not a new technique to adopt at this stage, but it surfaces pass-inventory detail this
project's own findings haven't captured yet.

## What the fix reveals about which render passes are hard

- **Water reflections remain incompletely fixed even in this mature, published fix** — the author's
  own comment: *"water reflections are still at wrong depth."* Worth expecting the same problem in
  this project's own stereo override unless specifically addressed — water reflection rendering very
  plausibly uses its own camera/projection setup (a mirrored or secondary view) distinct from the
  main-pass perspective registers already found (c28 in gameplay, c71/c75/c87, per this project's own
  recon), similar in spirit to how the shadow cascade turned out to be a separate camera-shaped
  register block that had to be excluded from the main override.
- **Shadow quality interacts badly with the stereo fix**: the fix's recommended settings are
  explicitly **"Water = high and Shadows = Medium,"** with higher shadow quality causing flickering
  artifacts, and safe houses/weapon depots specifically flagged as trouble spots for shadow depth
  artifacts. This is a strong, independent corroboration that this project's own shadow-cascade
  finding (the c12/c36/c16 registers, correctly excluded from the main perspective override as an
  orthographic sun-shadow rig, not the eye camera) is a real, non-trivial complication other stereo
  implementations for this exact game also had to navigate carefully — worth treating shadow-quality
  settings as a real variable to test against this project's own override once visual artifacts are
  being evaluated, not just a graphics-options footnote.
- **Fire and smoke particle effects needed dedicated, effect-level handling** ("profile-based
  approach," per the fix's own description) rather than falling out of a general camera override —
  worth flagging as a probable additional pass-inventory item alongside water reflections.
- **Crosshair/HUD needed explicit repositioning** to the correct screen depth — the same
  UI-depth-separation problem already familiar from this portfolio's other stereo-3D findings
  (Mad Max, Alice: Madness Returns, Prince of Persia 2008) — expected, not surprising, but worth
  having on record for this project's own eventual HUD-handling work.

## Why this matters for this project specifically

None of this changes the current roadmap (AER sync parity → matched-FOV virtual screen → head
tracking → GPU-only path) — but it flags **water reflections and fire/smoke effects as likely
needing their own investigation later**, the same way the shadow cascade did, rather than assuming
the already-found perspective registers (c28/c71/c75/c87) cover every render target that needs
per-eye treatment. Worth a note in the dossier's dead-ends/open-risks section so this isn't
rediscovered from scratch as a surprise once those effects are visually evaluated in the headset.

## Concrete next step

No immediate action on the current roadmap — file this as a known future investigation item (water
reflection camera/projection registers, fire/smoke effect handling) for whenever visual QA in the
headset surfaces artifacts in those specific effects, consistent with how the shadow cascade was
originally found and diagnosed.

## Sources

- https://helixmod.blogspot.com/2013/01/far-cry-2-dx9.html
