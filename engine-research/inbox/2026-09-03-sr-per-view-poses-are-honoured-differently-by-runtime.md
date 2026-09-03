# Per-view poses: the API question is settled, the RUNTIME question is not — and it varies

*Dropped by `/sr`, 2026-09-03. Create-only; fold into `ENGINE-DOSSIER.md` and delete.*

**Read this alongside the `/gr` drop already sitting in this same inbox**
(`2026-09-02-gr-openvr-1253-still-open-openxr-is-the-per-eye-pose-route.md`). That one establishes
the route; this one qualifies it. Fold them together rather than one after the other.

## Why it matters here

This project's AER submission needs the runtime to accept **two different poses in the same frame**.
OpenVR cannot express that (issue #1253, LukeRoss00, open seven years). OpenXR's
`XrCompositionLayerProjection` **can** — each `XrCompositionLayerProjectionView` carries its own
`pose` and `fov`, and they are submitted together in one layer, so there is no last-submit-wins
collision. That much is a **specification fact**, verified against Khronos's published header
`[verified-static 2026-09-02]`, and it is already in the cross-engine library.

**What is not settled is whether a runtime acts on them**, and the public record is worse than
"untested" — it is *contradictory*, which is the actual finding.

## Two public reports, three years apart, naming opposite culprits

`[reported 2026-09-03]` — both read online this sweep, nothing taken.

| when | who | what they report |
|---|---|---|
| 2020-10-29 | **LukeRoss00**, on Valve's [SteamVR discussion board](https://steamcommunity.com/app/250820/discussions/8/3001046778344834329/) | Submitting the spec-correct per-view poses from `xrLocateViews` through `xrEndFrame` gave a **wrong stereo baseline and a vertical offset between the eyes** on a Valve Index (SteamVR 1.15.4, OpenXR runtime 0.1.0). Workaround: submit **the head pose for both views**, and swap the two views' `fov.angleDown`. He recorded Oculus and Microsoft runtimes as correct. No reply on the thread. |
| 2023-09 | **SirKandela** (Chaos LTD), on the [Khronos forums](https://community.khronos.org/t/oculus-runtime-ignores-projection-layer-views-pose/110078) | The **Oculus desktop runtime appeared to ignore** the submitted projection-view pose, holding both projections on the HMD — and **SteamVR respected it**. **Rylie Pavlik** replied that a runtime truly ignoring the pose could not reproject correctly at all, so "ignored" may misdescribe what was seen. Reporter moved to a quad layer. |

**The pair is the point.** Per-view pose handling is **runtime- and version-specific and has
changed**. Do not inherit a verdict from a sibling project, a forum post, or this note — including
the encouraging half.

## The part that should change this project's test design

The natural experiment is *"submit two views with deliberately opposite offsets and see if the image
splits"*. It is weaker than it looks. Khronos's own reference text says `pose` and `fov` *"should
almost always derive from"* the values `xrLocateViews` returned, so a synthetic offset is exactly the
off-the-beaten-path submission a runtime is least likely to handle well — and a **null result would
be ambiguous** between "this runtime collapses per-view poses" and "this runtime declined an
implausible pose".

**The stronger acceptance test is LukeRoss's failure signature, run with the legitimate located
poses:** look for a visibly wrong **stereo baseline** *and* a **vertical misalignment between the
eyes**. Vertical disparity is the useful half — nothing in a correct stereo pair produces it, so
seeing it is a positive identification of the defect rather than the absence of an expected effect.

⚠️ If you hit it, **his workaround is also his diagnosis**: submitting the head pose for both views
(plus the `angleDown` swap) is a documented way to get a correct picture out of a runtime that
mishandles per-view poses — worth knowing before concluding the whole AER-over-OpenXR route is dead.

## One headset run answers this for two projects

`XIII2003-vr` has already **built** a projection-layer submission path with a deliberate-offset test
compiled in behind `[VR] OpenXrProjection` / `OpenXrProjectionTestOffsetM`, and is blocked on the
identical question. Whichever project gets to a headset first should record the runtime name **and
version** with the result, because per the above that is the thing the result is actually about.

## Confidence

`[reported 2026-09-03]` throughout. Two first-hand developer accounts on forums, on runtime versions
now years old, neither reproduced here. This moves the risk from *"untested"* to *"known to vary
between runtimes"* — it does not say what today's runtime does on your headset.

Generalised form now in the cross-engine library:
`docs/techniques/README.md` → "OpenXR carries a pose per view where OpenVR collapses to one" →
"⚠️ Expressible is not honoured".
