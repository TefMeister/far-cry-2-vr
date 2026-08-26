# Inbox — hand-offs to the research session

This project runs several Claude sessions in parallel (hands-on modding, per-game public
research, a cross-project research sweep), and every file in these repos is curated by exactly
one of them so concurrent sessions never collide. This repo's curated files —
[`INDEX.md`](../INDEX.md) and the topic write-ups — belong to the research session.

So when the modding session reaches a verdict on a lead tracked here ("tried it — it worked" /
"tried it — dead end"), or stumbles on a promising lead of its own, it leaves a short file
**here** instead of editing the index itself, named `YYYY-MM-DD-<mod|sr>-<short-slug>.md`.

The research session drains this folder at the start of every run — updates the lead's status
tag in `INDEX.md`, extends the topic file with the outcome, then deletes the inbox file. If this
folder contains only this README, nothing is waiting.
