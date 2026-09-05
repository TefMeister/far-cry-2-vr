Supersedes: `far-cry-2-vr/external-research/topics/2026-09-02-the-steamvr-per-eye-pose-bug-is-still-open-and-openxr-is-the-way-around-it.md`, lines 16 and 62, the "last activity 2019-11-23" claim

# The OpenXR topic still says 2019-11-23 — the correction reached its INDEX row but not the topic itself

Filed by `/gs`, 2026-09-05, home PC. Read-only sweep; nothing was edited. For `/gr`, which owns
`external-research/`.

## What happened

On 2026-09-04 `/gr` correctly established that `2019-11-23` is OpenVR #1253's **creation** date and
that its **last activity is 2020-04-22** `[verified-live 2026-09-04, n=1 API read]`, and updated
`external-research/INDEX.md` accordingly — the row now reads *"corrected + extended 2026-09-04:
last activity is 2020-04-22, not the 2019-11-23 creation date"*.

**The topic page it points at was not updated.** It still says:

- line 16 — *"report, **2019-11-23** — nearly seven years. `[reported 2026-09-02]`"*
- line 62 — *"re-checked 2026-09-02: still open, last activity 2019-11-23, no Valve response"*

So the index row and the topic it links to now disagree, and the topic is the one a reader lands on
after following the link.

## Why this is worth a drop rather than a shrug

The pattern is more interesting than the error: **the correcting lane updated the summary and left
the source page stale.** An index row is a pointer; a reader who follows it gets the uncorrected
text, and the `[reported 2026-09-02]` tag on line 16 makes it read as freshly checked. Nothing about
the estate's mechanics catches this — `/gs` check 2 found it only because a `Supersedes:` header
happened to name a *doc section*, and check 3b did not fire because `[reported]` is a perfectly
valid tag. It was simply pointed at a fact that has since changed.

## What is NOT wrong

The topic's conclusion stands, and so does everything built on it. #1253 is **still open**, still
has **no Valve response**, and ~6 years untouched is as fixed a constraint as ~7. The AER
shared-pose design in `far-cry-2-vr` is unaffected. This is a factual detail and a
consistency problem, not a design problem.

## Also worth folding in while you are there

The 2026-09-04 drop's second half — the **reporter's own partial fix (2019-12-17), lighthouse-driver
only, hedged, never in a changelog** — is in the INDEX row but not in the topic either. That is the
more substantive half: "no Valve response" is true, "never fixed" is not quite.

A parallel drop naming the dossier's copy of the same clause has been filed in
`far-cry-2-vr/engine-research/inbox/` for the modding lane. `XIII2003-vr/external-research/INDEX.md`
mentions correcting the same off-vocabulary tag in its own OpenXR topic — worth a glance to confirm
that one really was corrected in the topic and not just noted in the index.
