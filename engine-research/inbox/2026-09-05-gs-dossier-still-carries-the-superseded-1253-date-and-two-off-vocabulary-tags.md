Supersedes: `far-cry-2-vr/engine-research/ENGINE-DOSSIER.md` §AER stereo block, the clause "last activity 2019-11-23" (already superseded by the 2026-09-04 `/gr` drop, still uncorrected here)

# The dossier still says "last activity 2019-11-23", and carries two `[verified-static]` tags

Filed by `/gs`, 2026-09-05, home PC. Read-only sweep; nothing was edited.

## 1. The superseded date is still live in the dossier

`/gr` filed `inbox/2026-09-04-gr-1253-last-activity-is-2020-and-the-partial-fix-is-lighthouse-only.md`
(still undrained, 0d — so this is *pending*, not neglected). It corrects:

> **`2019-11-23` is the creation date.** Last activity is **2020-04-22**, read from the GitHub API.

`ENGINE-DOSSIER.md:382` still reads:

> **re-checked 2026-09-02: still open, no Valve response, last activity 2019-11-23** `[reported 2026-09-02]`

**The load-bearing conclusion is unaffected**, and that is worth saying first so nobody re-opens the
design: the dossier concludes *"a seven-year-old untouched defect is a fixed constraint of the
OpenVR submission path, so the current design is correct"*. At 2020-04-22 it is ~6 years untouched
instead of ~7. **Still open, still no Valve response, still a fixed constraint.** The AER
shared-pose design stands.

**The part that is more than a date:** the `/gr` drop also reports a **partial fix from the reporter
himself (2019-12-17), lighthouse-driver only, hedged, never in a changelog**. The dossier's
"no Valve response" is true but now incomplete — there *was* a reported partial fix, just not one
this estate can test (Quest 3 over Virtual Desktop, not lighthouse). That nuance is missing here and
is the more useful half to fold in.

**Where each copy lives**, since a correction has to chase all of them:

| file | state |
| --- | --- |
| `external-research/INDEX.md` | ✅ corrected 2026-09-04 (`/gr`, its own lane) |
| `engine-research/ENGINE-DOSSIER.md:382` | ❌ **still wrong** — modding, this drop |
| `external-research/topics/2026-09-02-…-openxr-is-the-way-around-it.md:16` and `:62` | ❌ **still wrong** — `/gr`'s lane, separate drop filed in `external-research/inbox/` |

## 2. Two off-vocabulary confidence tags in modding-owned files

Both are `[verified-static …]`, which is **not** one of the eight names, so it reads as a strong
claim to a human and counts as **untagged** to every tool:

- `engine-research/ENGINE-DOSSIER.md:387` — `[verified-static 2026-09-02, against the Khronos header]`
- `modding-notes/2026-09-04-aer-stereo-is-built-one-texture-per-eye-with-a-parity-test.md:55` — same tag

Both describe the same thing: reading a fact directly out of a shipped header. The vocabulary's
nearest fits are `[verified-numerically]` (if the check was arithmetic) or `[inferred-static]` (if
it was a reading). Given it is "the Khronos header says `XrCompositionLayerProjectionView` carries a
pose per view", `[inferred-static 2026-09-02]` understates it slightly and
`[verified-numerically]` overstates it — worth a moment's thought rather than a mechanical swap.

⚠️ **This is not an isolated slip.** `[verified-static]` appears in **seven** places across five
repos and three lanes. See the `/gs` report for 2026-09-05: it looks like a genuine gap in the
vocabulary ("I read this straight out of a shipped artefact and it is a fact, not an inference")
rather than seven independent mistakes. Fixing these two is still right; a convention decision is
the user's call, not this drop's.
