# OpenVR #1253 re-read 2026-09-04: your dossier's "last activity 2019-11-23" is out by five months, and a PARTIAL fix was reported for the lighthouse driver only

Filed by: `/sr`, 2026-09-04 (web half of the cross-engine sweep). For `/gr` to verify and fold into
`external-research/`. The dossier line this corrects is in the AER stereo block added today:
*"re-checked 2026-09-02: still open, no Valve response, last activity 2019-11-23"*.

## What the issue actually says, read today

<https://github.com/ValveSoftware/openvr/issues/1253> — *"VRTextureWithPose_t::mDeviceToAbsoluteTracking
not working correctly with alternate eye rendering"* `[reported 2026-09-04]`

| | |
| --- | --- |
| State | **Open** (never closed) |
| Opened | 2019-11-23 |
| **Last activity** | **2020-04-22** — a community request for a team update |
| Comments | 8, all from community members |
| Valve response | **None**, confirmed across the whole thread |

**So two of the three claims stand and one does not.** Still open: correct. No Valve response: correct,
and worth keeping because it is the load-bearing part. **Last activity 2019-11-23: wrong** — that is
the *creation* date; the thread ran into April 2020.

The practical effect on the argument is small but real. "Untouched since the day it was filed" is a
stronger-sounding claim than "untouched for six years since the last community bump", and the second is
the true one. Since the dossier uses the issue's age to justify treating one shared pose as a **fixed
constraint rather than a shortcut**, it is worth the claim being exactly right.

## ⚠️ The part that is new, and matters more

**A partial fix was reported in the thread and never appeared in a public changelog.** The filer
(LukeRoss00, the author of the AER technique) reported in December 2019 that the underlying bug had
been fixed in a SteamVR beta **for the lighthouse driver only** — not for the Oculus or WMR backends
`[reported 2026-09-04]`.

That changes what a passing test would mean for this project:

- **A "one shared pose is a constraint" design is still correct**, because it must hold on every
  runtime the mod could meet, and no cross-runtime workaround has ever been posted in that thread.
- **But a live test on an Index-class headset could pass** and would then say nothing about the Oculus
  or WMR path. That is a worse trap than a clean negative: it looks like the constraint has lifted.
- If a future session ever measures per-eye poses working on one headset, **record the runtime and the
  driver name beside the result**, and do not promote it to "OpenVR can do this".

Nothing here argues for changing the built design — it argues for the claim beside it being precise,
and for one line of caution against a test that could mislead.

## Suggested action

1. Correct the last-activity date where the dossier states it.
2. Add the lighthouse-only partial-fix report as `[reported 2026-09-04]`, with the caution about a
   single-headset test.
3. The cross-engine form of both points is already in the library's
   [OpenXR carries a pose per view](https://github.com/TefMeister/flat-to-vr-cross-engine-research/blob/main/docs/techniques/README.md#openxr-carries-a-pose-per-view-where-openvr-collapses-to-one)
   section, so nothing needs to be written up twice.
