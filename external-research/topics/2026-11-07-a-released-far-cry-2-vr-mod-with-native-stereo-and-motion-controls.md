# A released Far Cry 2 VR mod with native stereo and motion controls (Braxus, FarCry2VR)

Found 2026-11-07 (`/gr` estate sweep, CHECK-IN). Source: <https://github.com/Braxusx/FarCry2VR> (README and
release notes, read through GitHub's own API; nothing downloaded).

## What it is

- A third VR project for Far Cry 2, separate from the two the watch list already names (phunkaeg's FarCry2-vr and
  MonsterDeadWood's FC2VR). Whether it is related to MonsterDeadWood's FC2VR is not stated anywhere we read
  `[hypothesis]`: the README credits only "Braxus", made with Claude.
- Repo created 2026-10-22; first release v1.0.0 the same day; stable v1.0.3.1 on 2026-10-25; test builds
  v1.0.4-test2 to test5 on 2026-11-06; last push 2026-11-07. GPL-3.0. 39 stars when read `[reported 2026-11-07]`.
- Steam and GOG builds from one DLL; **needs the game in DirectX 10 mode** (the README says DX9 does not work) and
  a 32-bit OpenXR runtime (VDXR, or SteamVR 2.17+) `[reported]`.

## How it works, in its own words (our summary)

- Native stereo: the engine renders each eye itself with the headset's off-axis projection, not a reprojection.
- **One wide culling camera shared by both eyes, and sun shadows shared between the eyes**; the README says that is
  what brings busy towns to a steady 90 fps `[reported]`. Relevant to our dossier's culling question (the vorpX
  developer's "the matrix alone cannot fix culling", topic of 2026-09-11): this mod answers it with a separate,
  wider culling camera rather than the eye matrices.
- Frame pacing at the headset rate, a float-precision fix and a hand stabiliser for steady arms `[reported]`.
- Full motion controls through the game's own gamepad input (so every menu and prompt works), per-weapon grip poses,
  two-handed holds, aim down sights by raising the gun, body holsters, physical machete, thrown grenades with an arc,
  map/GPS and monocular in the hand, optional room-scale walking, flashlight, HUD on floating panels, DLSS/DLAA via a
  64-bit helper process `[reported]`.
- A launcher that also limits the game to 16 CPU cores, "which the engine needs" `[reported]`.
- When the 32-bit runtime is missing, the game starts on the monitor and its log shows `xrCreateInstance` failing
  with -51 (runtime unavailable) `[reported]`; matches the /gs drop of 2026-10-08 about 32-bit OpenXR.

## What it means for us

This is a released, playable, motion-controlled Far Cry 2 VR mod, which is exactly the case Tefa's rule of
2026-09-23 covers. Our project is already paused for the other two; this one is further along than either and is
public. Building on it is allowed by licence (GPL-3.0) only by forking under the same licence; our no-copied-code
rule still applies to us `[inferred-static]`. Watch-list drop: `engine-research/inbox/2026-11-07-gr-watching-far-cry-2-braxus-fc2vr.md`.
