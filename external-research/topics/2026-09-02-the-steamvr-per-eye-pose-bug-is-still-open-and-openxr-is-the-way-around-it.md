# The SteamVR per-eye-pose bug is still open — and OpenXR is the structural way around it

**Status:** 🆕 new · **Priority:** high — the board's live `[PD]` item is the AER stereo submission
design, and this re-check of the bug that design works around says the constraint is permanent on
OpenVR and avoidable on OpenXR.

## Why this was re-checked

`topics/2026-08-24-aer-steamvr-ghosting-and-cpu-readback-techniques.md` records **OpenVR issue
#1253**: `IVRCompositor::Submit(..., Submit_TextureWithPose)` is supposed to let each eye carry its
own pose, but SteamVR keeps only the pose from whichever `Submit` call happened **last**, so an
alternate-eye design ghosts badly on the eye submitted first. The research rules say to re-check a
tracked source for what has changed rather than re-read it cold. So: has it moved?

**No. It is still open, with no Valve response and no recorded fix.** Last activity is the original
report, **2019-11-23** — nearly seven years. `[reported 2026-09-02]`

That is a real finding rather than an absence. A defect that has sat untouched for seven years is not
going to be fixed inside this project's timeframe, so **it is a fixed constraint of the OpenVR
submission path, not a bug to wait out.**

## The part worth noticing about who reported it

The reporter is **LukeRoss00** — the author of R.E.A.L., the mod family this portfolio's own library
documents as *the* alternate-eye-rendering implementation for modern AAA titles. The report describes
exactly the technique this project's board item is building: render one eye per frame, reuse the
other eye's previous frame through reprojection.

So the canonical AER implementer built this design, hit this wall, reported it, and it was never
fixed. **This project is not going to find a cleverer way through it on OpenVR.** Treating that as
settled saves an implementation round.

## What that leaves, stated as design options

| Option | What you get | What it costs |
| --- | --- | --- |
| **Submit both eyes together each frame with one shared pose** (the board's current plan) | no ghosting; the compositor reprojects both eyes from the same pose | the older eye is a frame stale in *content*, which is inherent to AER, but it is no longer stale in *pose* |
| Per-eye poses on OpenVR | — | **unavailable**; the runtime discards the first |
| **Submit through OpenXR instead** | OpenXR's projection layer carries **a pose per view**, so each eye can legitimately carry its own | a second submission path to build and verify |

The third row is the one this re-check adds. OpenXR's composition-layer model is per-view by
construction rather than per-submit-call, which is why the collision that produces #1253 does not
arise in the same shape. **This is a structural argument from the API's design, not a tested result**
— `[hypothesis]`, and it should be verified against the specification before anyone builds on it. But
it is the first reason found to prefer OpenXR here for a reason other than portability.

**A cross-project note that makes it cheaper than it sounds:** `XIII2003-vr` already built an OpenXR
quad-layer host, recorded in its dossier §7 as **unverified on hardware**. Two projects on this
account now have a reason to want the same OpenXR submission path proven once.

## Concrete next steps

1. Keep the current plan — both eyes submitted together, shared pose, double-buffered textures. It is
   correct for OpenVR and this confirms there is no better option there.
2. Before building a second path, **read the OpenXR projection-layer specification** and confirm the
   per-view pose is honoured independently. One document, no launch.
3. If it is, note it as the reason to finish and verify an OpenXR host — and coordinate with XIII,
   which has one already written and never run.

## Sources

- https://github.com/ValveSoftware/openvr/issues/1253 — re-checked 2026-09-02: still open, last activity 2019-11-23, no Valve response
- This project's own `topics/2026-08-24-aer-steamvr-ghosting-and-cpu-readback-techniques.md`
- `XIII2003-vr/engine-research/ENGINE-DOSSIER.md` §7 — the unverified OpenXR quad-layer host

---

## ✅ VERIFIED the same day — the OpenXR half is no longer a hypothesis

This topic asked that the per-view pose claim be checked against the specification before anyone
built on it. **Done, from Khronos's published `openxr.h`** `[verified-static 2026-09-02]`:

- `XrCompositionLayerProjectionView` carries its **own `pose`** (`XrPosef`) and its **own `fov`**
  (`XrFovf`), alongside its `subImage`.
- `XrCompositionLayerProjection` holds an **array** of those views (`viewCount` + `views`), submitted
  **together in one layer, in one `space`**.

So there are no separate per-eye submission calls, and the last-call-overwrites-the-first collision
that produces OpenVR #1253 has nowhere to occur. **Per-eye poses are expressible in OpenXR by
construction**, and row three of the table above is now a verified option rather than a lead.

**The remaining risk is narrower, and worth stating precisely:** the API allowing independent
per-view poses is not the same as a runtime honouring them during reprojection, and SteamVR's OpenXR
runtime shares a vendor with the unfixed OpenVR defect. That is `[reported]`, and it is an empirical
question a headset session settles in one test — submit two views with deliberately different poses
and see whether both are honoured.

**Cross-project:** `XIII2003-vr` is the project that can run that test cheapest, since it already has
an OpenXR host written. Its own follow-up found something relevant here too — a **quad** layer has a
single pose and cannot carry stereo at all, so any OpenXR stereo submission must use the
**projection** layer. Full write-up:
`XIII2003-vr/external-research/topics/2026-09-02b-openxr-carries-a-pose-per-view-and-the-existing-host-is-the-wrong-layer-for-m2.md`.
