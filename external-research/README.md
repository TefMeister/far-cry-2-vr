# far-cry-2-vr — `external-research/`

Ongoing **public research** findings for the Far Cry 2 VR mod — leads, prior art, and technique write-ups gathered from publicly available sources (blogs, forums, existing tools, documentation), kept **separate from hands-on modding work**.

This repo exists so a dedicated research-only session can run *at the same time* as active reverse-engineering/coding work without any risk of the two colliding — research never writes to any of the other five repos, and the modding side just reads this one when it wants to check for new leads. See [INDEX.md](INDEX.md) for the running list of topics.

## The folders for Far Cry 2 VR

Everything for this game lives in one repository, one folder per job — so you
always know where to look. You are in **`external-research/`**.

| Folder | What lives here |
| --- | --- |
| [`mod/`](../mod/) | The mod itself — releases and install instructions. |
| [`dev-archive/`](../dev-archive/) | Full development history — snapshots, probes, dead ends, raw recon. |
| [`modding-notes/`](../modding-notes/) | Readable field notes / progress ledger. |
| [staging/far-cry-2-vr](https://github.com/TefMeister/staging/tree/main/far-cry-2-vr) 🔒 | **Private** — unverified WIP builds, cross-machine handoff. |
| [`engine-research/`](../engine-research/) | Distilled engine reference (dossier) + reusable VR RE playbook. |
| **`external-research/`** ← you are here | Ongoing public-research leads — read-only input to the other five, never the other way around. |

## How this repo is used

- A **research session** only ever reads the other five repos for context (to know what's already been tried) and only ever writes here.
- A **modding session** (live debugging, coding, testing) reads this repo whenever it wants to check for new leads, and folds anything useful into `-dev-archive`/`-modding-notes`/the code itself — attributed back to the topic file here.
- [INDEX.md](INDEX.md) is the front door: every topic, with a status tag, newest first.
- `topics/` holds one self-contained file per lead — not chronological session logs (that's what `-dev-archive`/`-modding-notes` are for).

## Credits

See [CREDITS.md](CREDITS.md) — every source, tool, and prior research this project builds on, credited by name, plus a standing notice on respecting creators' wishes.

## Contributing & policy

See [CONTRIBUTING.md](CONTRIBUTING.md) — how we credit and link sources, our
**study-everything-public but write-our-own-code** rule (we copy no one else's
source code or files, any license or price), the terms for reusing our work
(free, with credit), and how to request a correction or removal.
