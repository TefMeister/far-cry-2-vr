# OpenVR #1253 re-checked: still open after seven years — so per-eye poses are an OpenXR question, not an OpenVR one

Filed by: `/gr`, 2026-09-02
Topic: `external-research/topics/2026-09-02-the-steamvr-per-eye-pose-bug-is-still-open-and-openxr-is-the-way-around-it.md`
Dossier sections: §9/§12 (the AER submission risk), and the board's `[PD]` AER item

- **Re-checked 2026-09-02: OpenVR #1253 is still open, no Valve response, last activity 2019-11-23.** `[reported 2026-09-02]` A seven-year-old untouched defect is a **fixed constraint of the OpenVR submission path**, not something to wait out. The current plan — both eyes submitted together with a shared pose, double-buffered textures — is correct and there is no better option on that runtime.
- **The reporter is LukeRoss00**, author of R.E.A.L., describing exactly the AER technique this board item implements. The canonical AER implementer hit this wall and it was never fixed; that is worth treating as settled rather than re-derived.
- **⭐ OpenXR's projection layer carries a pose per view**, rather than per submit call, so the collision that produces #1253 does not arise in the same shape — the first reason found to prefer an OpenXR submission path here for something other than portability. `[hypothesis]` from the API's structure; **verify against the specification before building on it** (one document, no launch).
- **Cross-project:** `XIII2003-vr` already has an OpenXR quad-layer host, its dossier §7 marks it **unverified on hardware**. Two projects now want the same path proven once.

Suggested dossier change: in §9/§12 replace "OpenVR issue #1253 (open, no Valve fix recorded)" with the dated re-check and the consequence — per-eye poses are unavailable on OpenVR by design-in-practice, and OpenXR is the route if they are ever needed.
