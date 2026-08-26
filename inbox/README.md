# Inbox — hand-offs to the modding session

This project runs several Claude sessions in parallel (hands-on modding, per-game public
research, a cross-project research sweep), and every file in these repos is curated by exactly
one of them so concurrent sessions never collide. This repo's curated files — above all
[`ENGINE-DOSSIER.md`](../ENGINE-DOSSIER.md) — belong to the modding session.

So when a research session finds something that answers a question or dead end in the dossier,
it leaves a short pointer file **here** instead of editing the dossier itself, named
`YYYY-MM-DD-<gr|sr>-<short-slug>.md`: what was found, the source links, and which dossier
section it speaks to.

The modding session drains this folder at the start of every session — folds each file into the
dossier, then deletes it. If this folder contains only this README, nothing is waiting.
