# §AER: #1253's last activity is 2020-04-22, not 2019-11-23 — and the reported partial fix is for a driver this estate cannot test

Filed by: `/gr` (estate sweep, third pass 2026-09-04), for the modding lane.
Supersedes: `ENGINE-DOSSIER.md` §AER stereo block, the clause "last activity 2019-11-23"
Topic: `external-research/topics/2026-09-02-the-steamvr-per-eye-pose-bug-is-still-open-and-openxr-is-the-way-around-it.md` (§"Corrected and extended 2026-09-04")

## The correction

The dossier's AER stereo block says of OpenVR issue #1253: *"re-checked 2026-09-02: still open, no
Valve response, last activity 2019-11-23"*.

**`2019-11-23` is the creation date.** Read from the GitHub API on 2026-09-04
`[verified-live 2026-09-04, n=1 API read]`:

| field | value |
| --- | --- |
| state | `open` |
| created | `2019-11-23T19:26:26Z` |
| **last updated** | **`2020-04-22T17:29:00Z`** |
| comments | 8 |

**Two of the three claims stand.** Still open: correct. No Valve or maintainer response: correct, and
verified comment by comment across all eight — that is the load-bearing part and it is unchanged.
Only the age wording needs fixing, from "untouched since it was filed" to "no activity since a
community bump on 2020-04-22".

## The addition, which matters more than the date

**The reporter — LukeRoss00, author of the AER technique this project borrows — reported a partial
fix on 2019-12-17** `[reported 2026-09-04]`:

> "Apparently this problem was fixed in one of the latest betas, but only for the lighthouse driver
> (not for the Oculus and WMR backends)."

Reiterated 2019-12-29 as *"to the best of my knowledge, this issue has already been fixed with the
lighthouse driver"*. Note the hedging, and note that it never reached a public changelog. It is a
second-hand observation about a 2019 beta, nearly seven years old; its truth *today* is unknown.

## ⚠️ The estate-specific reading, which reverses the obvious caution

The natural warning would be "a live test on a lighthouse headset could pass and say nothing about
other backends". **That trap cannot occur here.** All in-headset testing happens on the home PC's
**Quest 3 over Virtual Desktop (VDXR / SteamVR)** — not a lighthouse device
(`claude-memory/MACHINES.md`). So:

- **A failure on our hardware is the expected outcome** if the 2019 report was ever right and still
  holds. It is weak evidence, not a discovery, and it does **not** show the bug is universal — the
  one backend ever reported fixed is the one we cannot test.
- **A pass on our hardware would be the more informative result**, because ours is the path reported
  still broken.
- Either way, **record the runtime and driver beside any per-eye pose result**. VDXR and SteamVR are
  different runtimes and Virtual Desktop supplies its own driver.

## Suggested dossier change

In the AER stereo block: change the last-activity date to 2020-04-22; keep "still open, no Valve
response" exactly as is; and add one or two sentences carrying the lighthouse-only report as
`[reported]` plus the hardware note above. **The design conclusion does not change** — one shared
pose stays a constraint rather than a shortcut, because the mod must hold on every runtime it could
meet, and the OpenXR per-view route remains the structural way around it. VDXR being an OpenXR
runtime makes that route the natural one on this hardware rather than a fallback.

## Source

- https://github.com/ValveSoftware/openvr/issues/1253 — metadata and all 8 comments, read via the
  GitHub API on 2026-09-04. Raised by an `/sr` drop of the same date and verified here independently.
