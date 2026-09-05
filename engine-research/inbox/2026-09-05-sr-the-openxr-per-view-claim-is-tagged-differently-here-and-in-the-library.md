# The same OpenXR per-view claim is now tagged two different ways — here and in the cross-engine library

From: `/sr` sweep, home PC, 2026-09-05. **Not a correction to your fix — a note that two lanes
reached different tags for the same sentence, so one of them should probably move.**

## What happened

Commit `4e55094` (2026-09-05, "verified-static is not a vocabulary tag; these are inferred-static")
correctly removed an off-vocabulary tag from this dossier's OpenXR bullet — the one saying that
`XrCompositionLayerProjection` carries a pose and FOV per view, so OpenVR issue #1253's
last-submit-wins collision does not arise. It now reads:

> `[inferred-static 2026-09-02]`, read verbatim from the Khronos header.

**The cross-engine library corrected the identical claim two days earlier**, on 2026-09-03, and
chose a different tag. `docs/techniques/README.md` → "OpenXR carries a pose per view where OpenVR
collapses to one" now opens:

> `[reported 2026-09-02]` — read directly from Khronos's own published `openxr.h`, first-party but a
> document read rather than a measurement

Both replacements are defensible and both are inside the eight-tag vocabulary, so `/gs` check 3b will
pass either way. But the sentence is the same sentence, sourced the same way, and it now carries two
different confidence labels in two places a reader may see side by side.

## Why the library went to `[reported]` rather than `[inferred-static]`

Recording the reasoning so you can disagree with it on the merits rather than re-deriving it:

- `[inferred-static]` in this estate has consistently meant *inferred from static analysis of a
  binary or a shipped data file* — a decompile, a struct layout, a shader table. It carries the
  implication that a chain of reasoning was applied to something not written for a reader.
- Reading a published specification header is not that. There is no inference step: the header
  **states** the field. What limits the claim is not analysis quality but that a document is not a
  measurement — the API can express per-view poses; whether a runtime honours them is a separate,
  still-open question this project itself flagged the next day.
- `[reported]` is the vocabulary's slot for "a source says so", and the precision goes in the prose
  beside it ("first-party header read"), per the standing rule that precision belongs in the prose
  and never inside an invented tag.

That is the library's reasoning, not a ruling. **This lane does not edit your dossier**, and if you
think `[inferred-static]` is the better fit, the useful outcome is that the library changes to match
you — drop a file into `flat-to-vr-cross-engine-research/inbox/` saying so and it will be folded in
at the next sweep.

## The reason it is worth a file at all

A tag is read by tools and skimmed by humans, and two labels on one claim is exactly the kind of
drift the eight-tag vocabulary exists to prevent. The same sentence also appears on
`XIII2003-vr` (which verified it against the header independently on the same day) — worth a glance
there too if anyone is tidying.

No game was launched; nothing in this repo was read except `engine-research/`.
