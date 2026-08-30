# Research index

Every research topic gathered for this project, newest first. Each row links to a self-contained
write-up in `topics/`. Status tags:

- 🆕 **new** — found, not yet acted on by the modding side.
- 👀 **reviewed** — a modding session has read it and factored it into a decision, but nothing shipped from it yet.
- ✅ **incorporated** — directly led to a real change (code, a test, a note) in one of the other five repos; linked below.
- ❌ **dead end** — checked out, didn't pan out; kept for the record so it isn't re-investigated from scratch.

| Date | Topic | Status | Summary |
| --- | --- | --- | --- |
| 2026-08-25 | [HelixMod Far Cry 2 corroborates pass inventory](topics/2026-08-25-helixmod-far-cry-2-corroborates-pass-inventory.md) | 🆕 new | A real 3D Vision fix for this exact game corroborates the shadow-cascade finding (shadow quality must stay Medium, not High) and flags water reflections and fire/smoke as separate, still-imperfect trouble passes worth expecting in this project's own override. |
| 2026-08-25 | [SteamVR frame timing + vsync gotcha](topics/2026-08-25-steamvr-frame-timing-vsync-gotcha.md) | 🆕 new | Valve's own frame-timing guidance (WaitGetPoses cadence, recommended loop order) plus a directly relevant gotcha: the game's own D3D9 Present must not vsync to the desktop refresh rate, or it fights the compositor's cadence independent of the already-planned double-buffer fix. |
| 2026-08-24 | [Head tracking: OpenVR pose-matrix layout + coordinate conversion + Vireio alternative](topics/2026-08-24-head-tracking-coordinate-conversion-and-alternate-technique.md) | 🆕 new | OpenVR's official `HmdMatrix34_t` layout reference (row-major storage, column-major read, a transpose gotcha, "B from A" direction convention). Dunia (Z-up/Y-forward) vs OpenVR (Y-up/−Z-forward) needs an axis remap with no universal pre-built matrix found — recommends deriving the target basis from the game's own already-parsed matrix each frame (extending the project's existing self-deriving approach) instead of hardcoding a guessed permutation. Also: Vireio Perception's VRBoost solves generic head-tracking by masquerading HMD rotation as mouse input, sidestepping coordinate conversion entirely — a documented fallback technique, rotation-only. |
| 2026-08-24 | [AER/SteamVR ghosting + CPU readback & shared-surface techniques](topics/2026-08-24-aer-steamvr-ghosting-and-cpu-readback-techniques.md) | 🆕 new | SteamVR has a known bug that discards one eye's pose on alternate-eye `Submit` — validates the planned double-buffer-both-textures approach rather than racing per-eye pose timing. Also: the standard double/triple-buffer fix for `GetRenderTargetData` stalls, the official D3D9Ex→D3D11 shared-surface interop technique (with a keyed-mutex incompatibility gotcha specific to D3D9), the standard `W = 2·Z·tan(θ/2)` matched-FOV virtual-screen formula for the next roadmap item, and confirmation that some AER ghosting is an industry-wide known tradeoff, not a bug to chase. |

## How to add a topic

1. New file in `topics/`, named `YYYY-MM-DD-short-slug.md`.
2. One row added to the table above, newest at the top.
3. Update the status tag here as it moves through review → incorporated/dead-end (the modding side should update this when it acts on a lead, so the index reflects reality without the research side needing to poll).
