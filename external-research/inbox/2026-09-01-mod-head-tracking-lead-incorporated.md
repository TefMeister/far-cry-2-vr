# Verdict on the head-tracking research lead: INCORPORATED, and the recommendation was right

**From:** the modding session, 2026-09-01
**For:** `/gr far-cry-2-vr` — please flip this row's status in `INDEX.md`.

## The lead

`topics/2026-08-24-head-tracking-coordinate-conversion-and-alternate-technique.md` — currently
tagged 🆕 **new**. **It should now be ✅ incorporated.**

## What it led to

Roadmap item 3 (head tracking) is built, in
`staging/far-cry-2-vr/proxy-winmm/src/{stereo.c,vr_bridge.c}`, and written up in
`modding-notes/2026-09-01-head-tracking-built-and-verified-numerically.md`. **Built clean, never
run** — there is no headset on the dev machine.

**The topic's central recommendation was followed directly and it was the right call.** Its §2 argued
against hardcoding an OpenVR→Dunia axis-permutation table, on the grounds that no published source
gives a verified matrix for this exact pairing, every real implementation hand-derives one, and a
wrong sign is invisible until someone is in a headset. It recommended extending the self-deriving
approach the eye offset already used.

The implementation derives the game's camera basis from `M`'s own rows every frame (`row0` right,
`row1` up, `row3` the view-depth axis) and reduces the entire conversion to one change of basis,
`B·H·Bᵀ`, with the single genuine convention difference (OpenVR's +Z is backwards) folded into `B`
by negating its forward column. There is no permutation table in the code.

The topic's §1 layout warning was also used as written: `HmdMatrix34_t` is stored row-major but read
as column-major, so the bridge extracts basis vectors from **columns** 0–2 and position from column
3, and the header records that explicitly. The `back`-not-`forward` naming in the new
`vr_bridge_hmd_pose()` API comes straight from that section's warning about sign errors.

## What the topic could not have predicted, and is worth knowing

The two bugs that actually appeared were **not** coordinate-convention bugs at all:

1. The camera-position solve was fed the *normalised* basis against the *raw* translation terms, so
   the projection's axis scales did not cancel and the recovered camera position was silently
   scaled.
2. The rotation was composed as the *camera* rotation, where what post-multiplies into `M` is the
   *world* transform — its inverse. That turns the view the right amount in the **wrong direction**.

Bug 2 matters for the research record: **it presents exactly like the handedness problem §2 warns
about.** Anyone debugging in a headset would reasonably reach for the handedness knob and "fix" it,
hiding a real inverse-composition error behind a plausible workaround. Both were caught by a
numerical harness (`proxy-winmm/tools/verify_head_tracking.py`) that checks the composition against
an independently rebuilt `P·V'` — agreement ~1e-15 after the fixes, versus a relative error of
**3.4** before.

If a future topic revisits head-tracking conversion for any project, the transferable lesson is that
"the world turns the wrong way" has at least two distinct causes with the same symptom, and the
cheap way to separate them is an offline numerical check rather than a headset session.

## The alternative in §3 (Vireio VRBoost input masquerade)

**Not adopted, not dead.** The matrix route was implementable and gives real 6DOF including
position, which mouse-input masquerade cannot express at all. Keeping §3 as the documented fallback
is right; no change of status needed for that part.
