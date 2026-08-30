# SteamVR's own frame-timing guidance: WaitGetPoses cadence, recommended loop order, and a vsync gotcha directly relevant to hooking the game's own Present

**Status:** 🆕 new · **Priority:** high — directly actionable for the in-progress "AER stereo — sync
wiggle parity with per-eye Submit" roadmap item, and surfaces a concrete gotcha the existing
CPU-readback/shared-surface research pass didn't cover.

## What was found

Valve's own developer documentation ("SteamVR/Frame Timing," Valve Developer Community wiki) and
related official/semi-official SteamVR developer guidance give the canonical, non-project-specific
picture of how an application should structure its frame submission relative to the compositor:

- **`WaitGetPoses` is the synchronization point** — called once per frame on the render thread, it
  blocks until roughly **3ms before the HMD's next vsync**, giving the freshest possible pose read
  before rendering. The recommended loop order is: **update game logic → `WaitGetPoses` → update
  player/camera state from the returned pose → render/Submit → `PostPresentHandoff`.**
- **The "scene" portion of a frame** (the part SteamVR's own frame-timing tools measure) is defined
  as the work done between `WaitGetPoses` returning and the *second* eye's texture being `Submit`ted
  — i.e. both eyes' rendering work counts as one measured unit, not two independent ones.
- **A concrete, directly relevant gotcha**: guidance explicitly warns to **disable vsync on any
  `Present` call to your own application window**, because otherwise the app synchronizes to the
  desktop monitor's own refresh rate (commonly 60Hz) instead of the headset's — and if a frame's GPU
  work runs long, it simply waits for the *next* vsync rather than the compositor's own cadence,
  producing exactly the kind of judder this project's own "vanilla-jank... uncapped FPS physics"
  observation already flagged as a pre-existing base-game issue, but as a *separate*, VR-bridge-side
  risk on top of that.

## Why this is directly relevant to the current roadmap item

This project's bridge captures the game's own D3D9 backbuffer via a `Present` hook and forwards it to
SteamVR. **The game's own D3D9 device's vsync/present-interval setting is a variable this project
should check and control explicitly**, separate from anything SteamVR itself does — if Far Cry 2's
own D3D9 device is presenting with vsync tied to the desktop refresh rate, that's a second, compounding
source of frame-pacing mismatch against the compositor's 90Hz cadence, on top of (not a replacement
for) the already-planned double-buffer decoupling between the game's alternating-eye capture cadence
and SteamVR's own Submit cadence (covered in the previous sweep's AER/readback topic). Worth checking
explicitly: what `PresentationInterval` the game's own device is created with, and whether forcing
`D3DPRESENT_INTERVAL_IMMEDIATE` (vsync off) on the hooked device improves or worsens observed judder,
independent of the AER double-buffering work already planned.

## Concrete next step

When implementing the AER sync-wiggle-parity work (current roadmap item 1), check and log the game's
own D3D9 device's presentation interval, and treat forcing it to immediate/no-vsync as a candidate
fix to test if judder persists after the double-buffered Submit pattern (from the previous research
pass) is in place — this is an independent variable from that fix, not covered by it.

## Sources

- https://developer.valvesoftware.com/wiki/SteamVR/Frame_Timing (direct fetch returned 403; content drawn from search-engine summarization of the same official page plus corroborating SteamVR developer forum discussion)
