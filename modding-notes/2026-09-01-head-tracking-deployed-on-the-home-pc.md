# 2026-09-01 — Head tracking is now installed on the headset machine; one launch decides it

**Date:** 2026-09-01, home PC (the Quest 3 machine), late evening. **Static only: the game was not
launched, nothing here has been run.** This is the follow-through to the dev PC's work earlier today
(`2026-09-01-head-tracking-built-and-verified-numerically.md`): that session built roadmap item 3
but had no headset; this one has the headset but does not launch games. So the deliverable is a
deployed build plus a precise test for the user.

---

## What was done

1. **Rebuilt the proxy from the current `staging/far-cry-2-vr/proxy-winmm/` source on this machine**
   with llvm-mingw (`clang -target i686-w64-mingw32 -O2 -shared -static …`, i.e. `build.ps1`'s
   command run from bash, since the script itself dies under PowerShell 5.1 on a harmless stderr
   warning — known since 2026-08-23). Result: `winmm.dll`, PE32/i386, 134,656 bytes, one warning
   (`EXTERN_C` redefined by the vendored `openvr_capi.h`; cosmetic). `[compile-verified 2026-09-01]`
2. **Re-ran the shipped-C numerical harness here, twice** — once compiled x86_64 like the dev PC
   did, once compiled **i686 -O2, the same target and optimisation level as the DLL itself**:

   ```
   SHIPPED C vs independent ground truth, worst relative error: 3.501e-07
   VERDICT: the C in stereo.c is CORRECT
   ```

   Identical to the dev PC's number, so the maths does not change with the toolchain or the
   target. `[verified-numerically 2026-09-01, n=200, i686 and x86_64]`
3. **Confirmed the DLL that was installed here was the pre-head-tracking build** before replacing
   it: `bin\winmm.dll` was dated 2026-08-23 17:42, 129,536 bytes, and contains **none** of the
   head-tracking log strings (`head tracking`, `handedness`, `camera-position solve FAILED`); the new
   build contains all of them. Both export the same 180 winmm forwarders and import the same eight
   DLLs, so the loader-facing surface is unchanged. `[measured 2026-09-01]`
4. **Deployed.** `C:\Steam\steamapps\common\Far Cry 2\bin\winmm.dll` is now the new build
   (SHA-256 prefix `c6bb446b18b332a4`). The old one is kept beside it as
   **`winmm.dll.2026-09-01-pre-headtracking.bak`** — to roll back, delete the new file and drop the
   `.2026-09-01-pre-headtracking.bak` suffix. `openvr_api.dll` (2026-08-17) was left alone.
5. **Checked the launch preconditions read-only.** The home `GamerProfile.xml` (Documents\My Games)
   still has `Platform="d3d9"`, `Fullscreen="1"`, render profile 1176×664 — exactly the configuration
   the bridge ran under on 2026-08-23, so nothing about the hook path has changed under us.

## Corrections to the record

- **`MACHINES.md` says Far Cry 2 is not installed on the home PC (checked 2026-08-21).** It is:
  `C:\Steam\steamapps\common\Far Cry 2\`, 3.5 GB, complete (`bin\FarCry2.exe`, `Dunia.dll`,
  `Data_Win32`), and it has been since at least 2026-08-22 — the 2026-08-23 headset session ran on
  it. That line is stale and is being corrected by the parent `/pd` session.

## The one test, and what each outcome means

Launch `bin\FarCry2.exe` **directly** (not via Steam while SteamVR is up — the Desktop Game Theatre
wedge noted 2026-08-21), get into gameplay, then:

| Step | Do | Good | Bad, and what it means |
|---|---|---|---|
| 1 | `Num0` (bridge) | image in the headset, as on 08-23 | no image → bridge regression, stop here; check `%LOCALAPPDATA%\FC2VR\fc2vr.log` for the OpenVR init lines |
| 2 | `Num7` (head tracking), turn your head | the world stays put as your head turns | nothing happens → log should show `stereo: head tracking on=1`; if it says `on=1` but `applied to 0 uploads`, no perspective uploads are being classified — a derivation problem, not a knob |
| 3 | still turning | direction correct | the world turns **with** you (inverted) or **mirrored** — only if genuinely mirrored press `Num2`. **Do not reach for `Num2` first:** the inverse-composition bug caught today presents exactly like this, and the harness says the shipped code is right, so an inversion here would point at the game's own camera fighting us, not at handedness |
| 4 | lean left/right | parallax looks natural | too little/too much → `Num1`/`Num3` (÷/×1.5); this IS a knob (unit scale, starting guess 1.0 = metres) |
| 5 | afterwards, read the log | `camera-position solve FAILED on N` with N small and **not growing** | N climbing steadily during gameplay → `row3` is not the forward axis for some perspective pass; the derivation in §6a needs revisiting before any tuning |

Everything in the table is still `[untested]` until the user has worn it.

## What is NOT established

- Nothing about the running game: whether the pose arrives, whether the composition survives the
  game's own camera logic, handedness, scale. The maths is verified; the game is not.
- The `EXTERN_C` redefinition warning has been there since the bridge landed and has never been
  looked at; it is harmless for a C translation unit but should be silenced properly some day.

🤖 Static work only; the game was not launched. One game file was replaced (our own proxy DLL, with a
dated backup); no game content was touched or committed.
