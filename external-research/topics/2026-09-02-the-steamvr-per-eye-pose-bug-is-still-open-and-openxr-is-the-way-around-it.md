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

**No. It is still open, and there is no Valve response anywhere in the thread.** The issue was
**opened** 2019-11-23 and its **last activity is 2020-04-22** `[verified-live 2026-09-04, n=1 API
read]` — untouched for over six years.

> ⚠️ **Read the "Corrected and extended 2026-09-04" section at the foot of this page before quoting
> anything above.** Two claims that stood here until 2026-09-05 were wrong: *"last activity is the
> original report, 2019-11-23 — nearly seven years"* conflated the **creation** date with the last
> activity, and *"no recorded fix"* was too strong — the reporter posted a **hedged, lighthouse-driver-only
> partial fix on 2019-12-17** that never reached a changelog. Both were corrected in that section on
> 2026-09-04, but this opening paragraph was left stale and contradicted it for a day; a `/gs` drop
> caught the mismatch on 2026-09-05 and it is fixed here now.

That is a real finding rather than an absence. A defect that has sat six years without a Valve
response, whose only reported fix is a hedged third-party observation about one driver this estate
cannot even test on, is not going to be fixed inside this project's timeframe — so **it is a fixed
constraint of the OpenVR submission path, not a bug to wait out.** Neither correction weakens that
conclusion, and the AER shared-pose design resting on it is unaffected.

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

- https://github.com/ValveSoftware/openvr/issues/1253 — re-checked 2026-09-02, and re-read in full
  via the GitHub API 2026-09-04: still open, **opened 2019-11-23, last activity 2020-04-22**, 8
  comments, no Valve response. (The 2026-09-02 reading of this line said "last activity 2019-11-23";
  that was the creation date — corrected 2026-09-04, and corrected here 2026-09-05.)
- This project's own `topics/2026-08-24-aer-steamvr-ghosting-and-cpu-readback-techniques.md`
- `XIII2003-vr/engine-research/ENGINE-DOSSIER.md` §7 — the unverified OpenXR quad-layer host

---

## ✅ VERIFIED the same day — the OpenXR half is no longer a hypothesis

This topic asked that the per-view pose claim be checked against the specification before anyone
built on it. **Done, from Khronos's published `openxr.h`** `[reported 2026-09-02]` — first-party, from the
header itself rather than from someone's description of it, but still a document read. (Tag
corrected 2026-09-03: `verified-static` is not one of the eight vocabulary names.):

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


---

## ✏️ Corrected and extended 2026-09-04 — one date was wrong, and a partial fix was reported that this estate's hardware cannot benefit from

Folded from an `/sr` inbox drop and **verified independently** against the GitHub API and the issue's
full comment thread on 2026-09-04.

### The date correction

This topic and the dossier both say *"last activity 2019-11-23"*. **That is the creation date, not the
last activity.** Read today `[verified-live 2026-09-04, n=1 API read]`:

| field | value |
| --- | --- |
| state | `open` |
| created | `2019-11-23T19:26:26Z` |
| **last updated** | **`2020-04-22T17:29:00Z`** |
| comments | 8 |

The load-bearing claims survive: still open, and **no Valve or maintainer response anywhere in the
thread** — confirmed comment by comment. Only the age wording changes, from "untouched since the day
it was filed" to "untouched for over six years since the last community bump".

### The part that is new: a partial fix, reported but never in a changelog

**LukeRoss00 — the reporter, and the author of the AER technique this project borrows — posted on
2019-12-17** `[reported 2026-09-04]`:

> "Apparently this problem was fixed in one of the latest betas, but only for the lighthouse driver
> (not for the Oculus and WMR backends)."

He reiterated it on 2019-12-29 as *"to the best of my knowledge, this issue has already been fixed
with the lighthouse driver"*. Note the hedges — *apparently*, *to the best of my knowledge*. This is a
**second-hand community observation about a 2019 beta**, not a changelog entry and not a Valve
statement, and SteamVR has had nearly seven years of change since. Treat its *current* truth as
unknown rather than as a fact about today's runtime.

### ⚠️ Why the obvious caution does not apply here, and what does instead

The natural warning is "a test on a lighthouse headset could pass and tell you nothing about other
backends". **On this estate that trap cannot occur, because there is no lighthouse headset.** All
in-headset testing happens on the home PC's **Quest 3 over Virtual Desktop (VDXR / SteamVR)**
(`claude-memory/MACHINES.md`) — not an Index-class device, and not the lighthouse driver.

That inverts the reading of a live result here:

- **A failure on this hardware is the expected outcome**, if the 2019 report was ever accurate and
  still holds. It would be **weak evidence**, not a discovery, and certainly not proof that the bug is
  universal — the one backend ever reported fixed is the one we cannot test.
- **A pass on this hardware would be the more informative result**, because it is the non-lighthouse
  path that was reported still broken.

Either way, **record the runtime and driver name beside any per-eye pose result** — VDXR and SteamVR
are different runtimes and Virtual Desktop supplies its own driver, so "it worked in the headset" is
not a statement about OpenVR in general.

### What does not change

The design conclusion stands untouched. One shared pose remains a **constraint rather than a
shortcut**, because the mod must hold on every runtime it could meet, and no cross-runtime workaround
has ever been posted in that thread. The OpenXR route — a `pose` and `fov` per view in one submitted
layer — remains the structural way around the whole question, and VDXR is itself an OpenXR runtime,
which makes that route the natural one on this hardware rather than a fallback.

### Sources for this section

- https://github.com/ValveSoftware/openvr/issues/1253 — issue metadata and all 8 comments, read via
  the GitHub API on 2026-09-04.
- `/sr`'s inbox drop of 2026-09-04, which raised both points; the cross-engine form is in the
  library's "OpenXR carries a pose per view" section.
- `claude-memory/MACHINES.md` for the home PC's headset and runtime.
