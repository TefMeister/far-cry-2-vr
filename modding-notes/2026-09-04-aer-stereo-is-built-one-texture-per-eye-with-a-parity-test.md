# 2026-09-04 (`/pd`, dev PC, static only) — AER stereo is built: one texture per eye, and a parity test that would catch swapped eyes

**The game was not launched, and nothing here has been run.** The board's `[PD]` row is closed, the
two inbox drops are folded in, and the dev PC's proxy is brought up to date (it was still the
2026-08-21 build, four versions behind).

---

## 1. What AER needed, and the one thing that could silently ruin it

The stereo override already draws **alternate frames from alternate eyes** (wiggle). The VR bridge,
until today, captured each frame and submitted the same mono texture to **both** eyes — throwing half
the stereo away and showing each eye the wrong view half the time.

AER is the fix: keep **one texture per eye**, write each frame into the texture for the eye it was
drawn with, and submit **both** every frame. Each eye then holds its own most recent view, at half
the frame rate, which is what Alternate Eye Rendering is.

**The thing that could ruin it is parity.** The eye and the image are produced in different places
inside one `Present` hook, in this order:

```
stereo_filter_upload()   during frame N   applies eye E_N
stereo_on_present()      end of frame N   flips the wiggle to E_(N+1)
vr_bridge_frame()        end of frame N   captures frame N's image
```

The bridge runs **after** the flip. Reading the wiggle state there would give `E_(N+1)` — the eye of
the frame about to be drawn, not the one being captured — and the eyes would be **swapped**. Swapped
eyes do not look like a bug. They look like working stereo with inverted depth, which is exactly the
kind of defect that survives a headset session and gets blamed on the maths.

So the value is **latched explicitly**: `stereo_on_present()` stores the completed frame's eye
*before* flipping, and the bridge reads `stereo_frame_eye()`. A frame where no per-eye offset was
applied at all — the override is off, or the game drew no perspective matrix — reports **0**, and the
bridge treats that image as mono and copies it to both eyes rather than assigning it to one.

## 2. The build

`staging/far-cry-2-vr/proxy-winmm/`, `[compile-verified 2026-09-04]`, 32-bit, 135,168 B:

- **`stereo.c`/`.h`** gain the latch and `stereo_frame_eye()`, with the ordering contract written
  into the header where the caller will read it.
- **`vr_bridge.c`** holds `g_tex11[2]` instead of one texture, copies each frame into the eye's own
  texture, and submits both — falling back to the other eye's texture until an eye has received a
  frame of its own, so the first frames of a run do not show one eye black (which reads as a broken
  bridge rather than a half-filled pipeline). A resize invalidates both eyes.
- The periodic log line now reports **left / right / mono** frame counts with what each pattern
  means, so a bad run is diagnosable from the log alone.

**Both eyes are still submitted in one `WaitGetPoses` cycle with one shared pose, and that is not a
shortcut.** OpenVR cannot express two poses in one frame — issue #1253, raised by the author of
R.E.A.L., the canonical AER implementation, **still open with no Valve response since 2019-11-23**
`[reported 2026-09-02, re-checked by /gr]`. OpenXR's projection layer does carry a pose per view
`[verified-static 2026-09-02, against the Khronos header]`, but SteamVR ships no 32-bit OpenXR
runtime and this is a 32-bit process, so it is not available here.

⚠️ And per the `/sr` drop, **expressible is not honoured**: two first-hand developer reports three
years apart name *opposite* runtimes as mishandling per-view poses. If per-eye poses are ever pursued
here, the acceptance test is LukeRoss's failure signature — a wrong stereo baseline **together with
vertical misalignment between the eyes**, since nothing in a correct stereo pair produces vertical
disparity — and his workaround (submit the head pose for both views, swap the views' `fov.angleDown`)
is also his diagnosis. Record the runtime **name and version** with any result.

## 3. The parity test, and the defect it caught in itself

`test/aer_parity_test.c`, run by `bash test/build-and-run-aer.sh` — **22 assertions, all passing**
`[verified-numerically 2026-09-04]`. It includes the **shipped `stereo.c`**, stubs only
`GetAsyncKeyState` (so it can press the hotkeys), `log_msg` and `vr_bridge_hmd_pose`, and drives a
simulated frame loop. It asserts:

- the override off ⇒ every frame reports mono and no offset is applied;
- in wiggle mode the latched eye **equals the sign of the offset actually written into the matrix**,
  read back from the bytes rather than from the flag under test, on every frame, and consecutive
  frames alternate;
- **the latch is taken before the flip** — the frame is drawn with the eye that was current when it
  started, `stereo_frame_eye()` reports *that* eye, and the wiggle state has meanwhile moved on;
- a frame with no perspective upload reports 0 rather than a stale eye;
- fixed left/right modes report that eye every frame.

⚠️ **The first version of this test passed while testing nothing.** The matrix it fed in was not
classified as perspective — the classifier reads the **bottom row's xyz** as the view-depth axis, and
the test had put a unit value in the wrong element — so no offset was ever applied, every reading was
0, and "the latched eye equals the applied sign" compared `0 == 0` and passed on all eight frames.
Only the assertions that demanded a *specific non-zero* eye failed, which is what exposed it. The
test now opens by asserting the matrix really is classified as perspective, and counts that all eight
frames actually got an eye. **A test whose assertions can be satisfied by "nothing happened" is worse
than no test**, because it reports success.

## 4. Deployment, and a correction about this machine

**Deployed to `Far Cry 2\bin\winmm.dll`** (135,168 B); the previous file is kept as
`winmm.dll.bak-2026-09-04-pre-aer` and one copy reverts.

⚠️ **That previous file was 129,536 B, dated 2026-08-21 — the pre-head-tracking build.** The
2026-09-01 entry recording head tracking as deployed was about the **home PC**; the dev PC was never
updated. So this deploy brings the dev PC forward by head tracking (2026-09-01) *and* AER (today) at
once. Head tracking is moot here — no headset on this machine — but it means the dev PC's binary and
the home PC's binary were two different versions until now, which is worth knowing before comparing
any behaviour between them.

## 5. What the next headset session answers

Still the home PC, still one launch. Launch `bin\FarCry2.exe` directly, then:

| step | outcome and meaning |
| --- | --- |
| `Num0` (bridge), `Num5` (stereo override on) | the log's `AER:` counters start moving |
| read the counters | **left and right within one of each other, mono not climbing** ⇒ parity is working. **Mono climbing** ⇒ no per-eye offset is reaching the frames — is the override actually on? **One eye stuck** ⇒ the wiggle is not alternating, so the mode is left or right rather than wiggle |
| look at the image | **depth that reverses when you press `Num8` twice** (left-only, then right-only) is the sign of correct AER; if depth looks inverted in wiggle mode but correct in a fixed mode, the eyes are swapped and §1's latch is the place to look |
| `Num7`, then turn your head | the head-tracking test from 2026-09-01, unchanged: world stays put = pass; world turns with you = read the log **before** pressing `Num2`, because a climbing `camera-position solve FAILED on N` means the derivation is wrong rather than the handedness |

**NOT established:** that any of this renders. The parity contract is proven and the build is proven;
no frame has ever been submitted from it.
