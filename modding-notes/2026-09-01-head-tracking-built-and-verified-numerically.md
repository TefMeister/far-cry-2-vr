# 2026-09-01 — Head tracking built; the maths verified numerically, not by eye

**Date:** 2026-09-01, dev machine. **The game was never launched** (a parallel session owns the
machine's one "game may run" slot), and there is no headset here. Roadmap item 3 is **built, built
clean, and never run.** Code in `staging/far-cry-2-vr/proxy-winmm/`.

---

## What was built

The VR bridge already called `WaitGetPoses` every frame purely to pace to the HMD, and threw the
result away. It now publishes the HMD pose (`vr_bridge_hmd_pose`) in **OpenVR's own space, with no
axis conversion applied**. `stereo.c` composes it into the per-frame view-projection.

New hotkeys, all defaulting OFF, all requiring the bridge (Num0) to be live:

| Key | Effect |
|---|---|
| `Num7` | head tracking on/off |
| `Num2` | flip head-rotation handedness — **try this first** if turning your head turns the world the wrong way |
| `Num1` / `Num3` | head position scale ÷ / × 1.5 |

## How it avoids a hardcoded axis table

`external-research/topics/2026-08-24-head-tracking-coordinate-conversion-and-alternate-technique.md`
made a specific argument: no published source gives a verified OpenVR→Dunia axis matrix, every real
implementation hand-derives one, and **a wrong sign is invisible until someone is in a headset** —
which on this project means a round trip to the other machine just to learn that a guess was wrong.
It recommended extending the self-deriving approach the eye offset already uses. That is what this
does.

* **The game's camera basis is read from `M`'s own rows** — `row0` = camera right, `row1` = camera
  up, `row3` = the view-depth axis (`is_perspective_vp()` already requires row3 to be a roughly
  unit world-space axis, which is what makes this safe). Nothing is assumed about Dunia's
  convention.
* **The one real convention difference** — OpenVR's +Z points backwards where the game's third axis
  points forwards — is folded into the basis by negating its forward column. The entire conversion
  is then a single change of basis, `R = B·H·Bᵀ`, with no permutation table anywhere.
* **The camera's world position is solved from `M`, not assumed.** For `M = P·V` the camera maps to
  the view origin, so `row0·c + m[3] = 0`, `row1·c + m[7] = 0`, `row3·c + m[15] = 0` — three
  equations, three unknowns, solved by Cramer. Row 2 is deliberately unused: it carries the
  near/far mapping and is the row most likely to be unusual.
* **No `P`/`V` split anywhere.** Every factor is on the right of `M`, i.e. in world space:
  `M' = M · [ R | c − R·(c + t) ]`.

## Two real bugs, caught by testing rather than by reading

Both would have shipped to the headset looking plausible.

1. **The camera-position solve was fed the *normalised* basis against the *raw* translation terms.**
   The projection's per-axis scales cancel only when both sides come from the same unnormalised row,
   so this silently scaled the recovered camera position by `sx`/`sy`. The symptom would have been a
   head rotation pivoting around the wrong point — a subtle, sickening error, not an obvious one.
   Fixed by solving with the raw rows.
2. **The rotation was composed as the *camera* rotation.** What post-multiplies into `M` is the
   matching transform of the **world**, which is its **inverse**. Composing the camera rotation
   directly turns the view by the right amount in the **wrong direction** — which in a headset reads
   exactly like a handedness problem and would have been "fixed" with the `Num2` knob, hiding a real
   error behind a plausible-looking workaround.

### The check that caught them

`proxy-winmm/tools/verify_head_tracking.py` transcribes the C algorithm line for line and compares
its output against an **independently constructed** `P·V'` — a matrix rebuilt from scratch for a
camera that has actually been rotated and translated. Over 300 random cameras, projections, head
rotations and head positions:

```
camera position recovery, worst relative error : 4.7e-16
full M' vs independent ground truth, worst rel : 9.5e-16
```

Before the fixes the second number was **3.4** — i.e. wrong by more than the matrix itself. Both
bugs fail the test loudly, which is the point: this is maths that cannot be checked cheaply in a
headset, so it is checked here instead.

Also fixed in passing: `key_edge()`'s rising-edge slot array was `static int prev[8]` and the new
keys index up to slot 8 — a one-past-the-end write on every frame a head-tracking key was polled.

## Status and honest limits

`[verified-numerically 2026-09-01, n=300 synthetic cases]` for the maths.
`[untested]` for everything about the actual game.

**Verified:** the algebra, against ground truth. **Not verified, and not verifiable here:**

* That `row3` really is the forward axis for *every* perspective upload FC2 makes. The classifier
  accepts anything with a roughly-unit bottom row; if some pass has a different meaning there, the
  solve produces a confident, wrong camera position. The periodic log now reports
  `camera-position solve FAILED on N` — **a steady non-zero count means the derivation is wrong for
  this game, not that a knob needs tuning.**
* Handedness. If the derived basis is the opposite handedness to OpenVR's, the rotation comes out
  mirrored. `Num2` switches to the inverse head rotation. This is a convention flip, not a different
  derivation — but see bug 2 above: **do not reach for `Num2` before confirming the direction is
  actually mirrored and not merely inverted.**
* Scale. Dunia units are ≈ metres (established 2026-08-21), so `g_head_scale = 1.0` is the honest
  starting guess, not a measurement.
* Whether the game's own camera logic fights the override.

### Next headset session, in order

1. Bridge on (Num0), confirm image. 2. `Num7`, look around — does the view track at all?
3. If it tracks backwards, `Num2`. 4. Lean — does parallax look right? Tune with `Num1`/`Num3`.
5. Check the log for `camera-position solve FAILED`.

🤖 Static work only; the game was not launched and no game file was modified.
