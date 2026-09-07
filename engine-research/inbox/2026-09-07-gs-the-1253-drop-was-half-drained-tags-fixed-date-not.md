Supersedes: `inbox/2026-09-05-gs-dossier-still-carries-the-superseded-1253-date-and-two-off-vocabulary-tags.md` §2 — its two off-vocabulary tags are now FIXED, so only §1 of that drop is still outstanding

# The 09-05 bundle was half-drained: the tags were fixed, the date was not — and the drop is still sitting in the inbox

Filed by `/gs`, 2026-09-07, home PC (twenty-fifth run). Read-only sweep; nothing was edited.

## What was measured

`[verified-numerically 2026-09-07]` — every line below was read out of the working tree at
`origin/main` today.

| the 09-05 drop asked for | state today |
| --- | --- |
| §2: retag `ENGINE-DOSSIER.md:387` off `[verified-static]` | ✅ **done** — now `[inferred-static 2026-09-02]` |
| §2: retag `modding-notes/2026-09-04-aer-stereo-is-built…:55` | ✅ **done** — that file now carries only `[compile-verified 2026-09-04]` and `[verified-numerically 2026-09-04]` |
| §1: correct `ENGINE-DOSSIER.md:382`, the clause "last activity 2019-11-23" | ❌ **still wrong**, verbatim unchanged |

`ENGINE-DOSSIER.md:382` still reads:

> **re-checked 2026-09-02: still open, no Valve response, last activity 2019-11-23**

The correct value, from the 2026-09-04 `/gr` drop reading the GitHub API, is **2020-04-22**, and
that drop also carries the more useful half — a **partial fix from the reporter himself
(2019-12-17), lighthouse-driver only, hedged, never in a changelog**, which the dossier phrase
"no Valve response" leaves out.

**The load-bearing conclusion is still unaffected**, exactly as the 09-05 drop said: ~6 years
untouched instead of ~7, still open, still a fixed constraint of the OpenVR submission path. The
AER shared-pose design stands. Nobody needs to re-open the design.

## Why this is worth a second drop rather than a nag

Two `/gs` drops naming the same date would be duplication. This one is not that: it is the
**first evidence that the bundle was opened and only partly acted on**, which age alone could not
show and which the drop file itself cannot show either — it is still present in the inbox, so by
the old reading it looked like "waiting for the owner to run".

That is the pattern `inbox/2026-09-07b-gs-bundled-drops-get-half-drained-and-nothing-shows-it.md`
(filed against `visceral-re2-vr` earlier today) describes. **This is a second, independent
instance of it, on a different project and a different lane** — so the pattern is no longer a
single observation `[verified-numerically 2026-09-07, n=2 projects]`.

It is also what the new check-1 `STALLED` flag was built to surface, and it worked: this row has
carried `<-- STALLED` since the flag shipped, on the correct grounds (the owner committed to
far-cry-2-vr files two days ago, after the drop landed).

## What the owner has to do

One line in `ENGINE-DOSSIER.md:382`, plus a clause for the 2019-12-17 partial fix, and then
**delete both this file and the 09-05 file by name** — they are finished together.

⚠️ The third copy of the wrong date, `external-research/topics/2026-09-02-…-openxr-is-the-way-around-it.md:16`
and `:62`, is in `/gr`'s lane and has its own drop. Not yours; do not edit it.

Lane: /gs
