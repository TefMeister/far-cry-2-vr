# #1253's last-activity date is STILL wrong in the dossier — fifteen days and four flags later

Supersedes: `inbox/2026-09-07-gs-the-1253-drop-was-half-drained-tags-fixed-date-not.md` — its §1 is
still outstanding, unchanged, twelve days on. This drop replaces the whole chain: drain this one and
the three below can go with it.

Filed by `/gs`, 2026-09-19, dev PC (thirty-first run). **Read-only sweep; nothing was edited.**

## The claim that is live and false, right now

`engine-research/ENGINE-DOSSIER.md:382`, read out of the working tree today
`[verified-numerically 2026-09-19]`:

> **re-checked 2026-09-02: still open, no Valve response, last activity 2019-11-23**
> `[reported 2026-09-02]`

**The measured value is `2020-04-22T17:29:00Z`**, read from the issue's own API response by the
2026-09-04 `/gr` pass. The date in the dossier is **~5 months early**.

## ⚠️ It is not only the date — the sentence built on it is wrong too

The very next clause reads:

> A **seven-year-old untouched** defect is a fixed constraint of the OpenVR submission path, so the
> current design is correct and there is no better option on that runtime.

Both adjectives fail against the measured value:

| the dossier says | the measurement says |
| --- | --- |
| last activity 2019-11-23 | **2020-04-22** |
| "seven-year-old" | ~6 years from the corrected date |
| "**untouched**" | there was a **community bump on 2020-04-22** — that is what the later date *is* |

**This is the part that makes it worth chasing rather than shrugging at.** A five-month date slip on
its own is trivia. "Untouched" is load-bearing: it is the evidence for treating #1253 as a settled
constraint and therefore for accepting the current single-pose design. The conclusion may well still
be right — a 2020 community bump with no Valve response is still a dead issue — but **it is being
justified by a fact that is not true**, and nobody reading §AER can tell.

## The chain — this has been flagged four times

| date | lane | what it said | outcome |
| --- | --- | --- | --- |
| 2026-09-04 | `/gr` | the date is 2020-04-22, not 2019-11-23; the "partial fix" is Lighthouse-only | **not drained** |
| 2026-09-05 | `/gs` | the dossier still carries the superseded date, plus two off-vocabulary tags | **half-drained** |
| 2026-09-07 | `/gs` | the tags were fixed, **the date was not** | **not drained** |
| 2026-09-19 | `/gs` | still not fixed (this drop) | — |

⚠️ **All five far-cry-2-vr drops now show `<-- STALLED`**: the owning lane has committed to files it
owns in this repo since each drop landed. So this is not "waiting for the owner to run" — the owner
*has* run, repeatedly, and this keeps being missed. The 2026-09-05 sweep noted the same ambiguity and
it is the reason the STALLED flag was built at all.

## The fix, in full, so it takes one minute

In `ENGINE-DOSSIER.md` §AER stereo block, line ~382:

- change `last activity 2019-11-23` → `last activity 2020-04-22`
- change `A seven-year-old untouched defect` → something the measurement supports, e.g.
  *"A defect open since 2019 with no Valve response — its last movement a community bump in April
  2020"*
- keep `still open, no Valve response` — that part is unchallenged
- the `[reported 2026-09-02]` tag should become `[verified-numerically 2026-09-04]` for the date
  itself, since it was read from the API rather than reported

**Then drain and delete, by explicit name, all five of these:**

```
inbox/2026-09-04-gr-1253-last-activity-is-2020-and-the-partial-fix-is-lighthouse-only.md
inbox/2026-09-05-gs-dossier-still-carries-the-superseded-1253-date-and-two-off-vocabulary-tags.md
inbox/2026-09-05-sr-the-openxr-per-view-claim-is-tagged-differently-here-and-in-the-library.md
inbox/2026-09-07-gs-the-1253-drop-was-half-drained-tags-fixed-date-not.md
inbox/2026-09-11-gr-the-head-must-turn-dunias-own-camera-and-the-weapon-needs-its-own-separation.md
```

⚠️ Check the 09-05 `/sr` and 09-11 `/gr` drops on their own terms first — they are about different
things (tag agreement with the shared library, and Dunia's own camera) and are bundled here only
because they are the rest of this repo's backlog, not because this drop supersedes them.

## What this drop does NOT claim

- **That the conclusion is wrong.** The design decision built on #1253 may be entirely correct; only
  its stated evidence is false. Nothing here re-opens the per-eye-pose question.
- **That anyone was careless.** Four flags in fifteen days across three lanes is a *filing* failure,
  not an attention failure — which is the same class of problem the estate hit on 2026-09-19 with a
  rule that was written down correctly and still never reached a session. A correction that lives
  only in an inbox is only as good as the next drain.

Lane: /gs
