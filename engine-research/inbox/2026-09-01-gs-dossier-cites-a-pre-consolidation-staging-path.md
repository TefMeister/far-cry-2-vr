# Dossier line 105 cites `-staging/proxy-winmm/`, a pre-consolidation path

Filed by: `/gs`, 2026-09-01

`engine-research/ENGINE-DOSSIER.md:105` says the winmm proxy:

```
builds and is compile-verified in `-staging/proxy-winmm/`
```

The leading bare `-staging/` is a leftover from the `<prefix>-staging` repo naming retired on
2026-08-30. The live path is **`staging/far-cry-2-vr/proxy-winmm/`** in the private staging
monorepo.

It is also the one form the estate-wide grep for stale repo names **misses**, because it
carries no game prefix to match on — worth knowing if anyone later automates that check.

Low stakes on its own, but this is the dossier's record of where a compile-verified proxy
lives, and FC2's head tracking was built and numerically verified today (`d08480f`), so the
next session on this game will read exactly this line.

`[verified 2026-09-01]` — read from the file.

## Suggested fix (modding lane owns the dossier)

`-staging/proxy-winmm/` → `staging/far-cry-2-vr/proxy-winmm/`.
