# The dossier still says SteamVR has no 32-bit OpenXR runtime

Supersedes: engine-research/ENGINE-DOSSIER.md, the "Expressible is not honoured" bullet near line 432 ("Not available here: SteamVR ships no 32-bit OpenXR runtime and this is a 32-bit process")

From `/gs`, 2026-10-08.

SteamVR 2.17 left beta on 2026-09-10 with "Added support for 32-bit OpenXR applications" `[reported 2026-10-07]`
(source and detail: `flat-to-vr-cross-engine-research/inbox/2026-10-07-gr-steamvr-now-ships-32-bit-openxr.md`,
and the Alan Wake dossier, which found `steamxr_win32.json` on the dev PC `[measured 2026-10-06]`).

So the reason given for "one shared pose for both eyes is forced" no longer holds as written: a 32-bit OpenXR
route to SteamVR now exists. Whether SteamVR honours per-view poses is still the separate open question the
same bullet names. Not yet run end to end for Far Cry 2.

Suggested change: reword the clause to "SteamVR 2.17+ ships a 32-bit OpenXR runtime (not yet tried here)", and
check `status/far-cry-2-vr.md` for the same reasoning. The dated modding notes from 2026-08-21 and 2026-09-04
are history and need no edit.

The library's copy in `docs/engines/dunia.md` (line 61) is reported to `/sr` separately.
