# SteamVR's known AER per-eye-pose bug (validates the double-buffer plan) + concrete D3D9 readback/shared-surface techniques for the next three milestones

**Status:** 🆕 new — one finding validates the existing plan and sets expectations, the rest are
concrete implementation techniques for the next three roadmap items.

The home-machine gate cleared 2026-08-23 (`first-real-headset-contact.md`); the plan for what's next
is explicit in that entry and `steamvr-bridge-works-first-public-release.md`: **(1) AER stereo — sync
wiggle parity with per-eye Submit, double-buffered D3D11 textures; (2) a sane virtual-screen/matched
projection instead of full-FOV stretch; (3) head tracking into the `M' = M·T(t)` override; (4) later,
a GPU-only shared-surface path to remove the CPU readback.** This pass covers concrete, well-trodden
public technique for (1), (2), and groundwork for (4); (3) is mostly project-specific math once (1)
lands and wasn't chased separately this pass.

## 1. SteamVR has a known, unresolved bug that discards one eye's pose in alternate-eye submission — this validates (doesn't complicate) the planned double-buffer approach

**OpenVR GitHub issue #1253** ("`VRTextureWithPose_t::mDeviceToAbsoluteTracking` not working correctly
with alternate eye rendering," `github.com/ValveSoftware/openvr/issues/1253`, open, no Valve fix
recorded) documents exactly the failure mode anyone doing AER-to-SteamVR needs to know about:

- `IVRCompositor::Submit(..., Submit_TextureWithPose)` is supposed to let each eye carry its own pose
  (necessary for true AER, where the two eyes are logically rendered at different moments). In
  practice, **SteamVR only keeps the pose from whichever `Submit` call happened last** — submit left
  then right, and only the right eye's pose is honored (the reverse order flips which eye is wrong).
  This is a compositor-level limitation, not something fixable from the application side.
- The reporter confirms this works correctly on the Oculus/Meta runtime (which has an explicit
  per-eye `RenderPose` field in its layer struct) — it's specifically a SteamVR architecture gap.
- Reported consequence: **severe ghosting** when an app tries to give each eye a genuinely distinct,
  independently-timed pose through Submit.

**Why this matters for FC2 specifically:** the plan already calls for "double-buffered D3D11
textures" rather than submitting a fresh pose-tagged frame the instant each eye's wiggle-frame is
ready — this finding explains *why* that's the right call, not just a nice-to-have. Given SteamVR
ignores per-eye pose timing anyway, there's no benefit to racing to Submit each eye the moment its
wiggle-frame lands; the correct pattern is to **hold the last-captured left and right frames in two
textures and Submit both every compositor cycle using the current single polled pose** (i.e., decouple
capture cadence — tied to the game's alternating render — from Submit cadence, tied to the
compositor's own frame timing via `WaitGetPoses`). This sidesteps the SteamVR bug entirely rather than
fighting it, and sets the right expectation going in: a small amount of eye-to-eye temporal offset
(one wiggle-frame's worth) is an inherent property of AER-to-SteamVR, not a bug to chase out.

## 2. GetRenderTargetData stalls are a known, solved problem — double/triple-buffer + deferred readback

The current v0.1 bridge does synchronous `GetRenderTargetData` readback in the Present hook (by
design, "safety first, speed later" per the release notes) — the natural next optimization before or
instead of the full GPU-only rewrite. This is thoroughly documented, non-project-specific D3D9
territory (Microsoft Learn's `IDirect3DDevice9::GetRenderTargetData` reference; multiple GameDev.net
threads on the exact same problem):

- `GetRenderTargetData` normally **stalls the calling thread** until the GPU finishes every pending
  command up to that point — full pipeline stall, not a background copy.
- The documented fix used broadly in this exact scenario (readback-heavy legacy-game tooling): **don't
  read back the frame you just rendered — read back a frame or two behind.** Allocate 2–3
  system-memory surfaces in rotation, issue the copy-to-sysmem request for the *current* frame, and
  only `Lock`/read the surface that was requested 1–2 frames ago. By the time you read it, the GPU has
  almost certainly already finished it in the background, so the lock returns immediately instead of
  stalling.
- Combine with a background worker thread doing the `Lock` + upload-to-D3D11 step off the Present
  thread entirely, so even the reduced/near-zero stall doesn't block the game's own frame pacing.
- This is presented as the standard technique (not attributed to one project) across the GameDev.net
  discussions found — general D3D9 knowledge, not proprietary to any single mod.

**Why this matters:** this is a much smaller change than the full D3D9Ex/D3D11 shared-surface rewrite
(item 4 on the roadmap) and could meaningfully cut readback latency/CPU cost in the meantime — worth
doing as an intermediate step, or skip straight to item 4 below if the team would rather not touch the
readback path twice.

## 3. D3D9Ex → D3D11 shared-surface interop: the official technique, and a sync gotcha specific to this pairing

For the eventual GPU-only path (roadmap item 4), Microsoft's own interop article is the canonical
reference: **"Surface Sharing Between Windows Graphics APIs"**
(`learn.microsoft.com/en-us/windows/win32/direct3darticles/surface-sharing-between-windows-graphics-apis`),
which explicitly covers D3D9Ex ↔ D3D11 sharing (this is a Windows-platform-documented, intentional
interop path — not a hack). Mechanism, confirmed across that doc plus GameDev.net discussion threads
on the same D3D9Ex/D3D11 pairing:

- Create the shared texture as a D3D11 resource with `D3D11_RESOURCE_MISC_SHARED` (or the keyed-mutex
  variant, see gotcha below), obtain its `HANDLE` via `IDXGIResource::GetSharedHandle`, then open that
  handle from the D3D9Ex side with `IDirect3DDevice9Ex::CreateTexture(..., pSharedHandle)`. The game's
  own D3D9Ex device can then render (or in this project's case, `StretchRect`/copy from its backbuffer)
  directly into the shared surface — no CPU round-trip.
- **Gotcha found, specific to this exact pairing:** `D3D11_RESOURCE_MISC_SHARED_KEYEDMUTEX` (the
  "clean" cross-API sync primitive normally recommended for shared surfaces) **is not understood on
  the D3D9 side at all** — there is no `IDirect3D9KeyedMutex`-equivalent interface. Keyed-mutex
  synchronization, the usual answer to "how do I know the other API is done with the shared surface,"
  is **not an option for a D3D9Ex producer**. The practical alternative (standard technique for this
  specific D3D9Ex/D3D11 gap, not this project's invention) is a `IDirect3DQuery9` event query (or
  simply a `Flush` + a frame of latency tolerance) on the D3D9 side to know when a copy has landed,
  combined with the same double/triple-buffering discipline as item 2 above so the D3D11/OpenVR side
  never reads a surface the D3D9 side might still be writing.
- Both APIs must be running on the **same GPU adapter** for the shared handle to open successfully —
  worth an explicit check/fail-safe in the plugin given the CPU-readback v1 never had to care about
  adapter affinity.

**Why this matters:** confirms the GPU-only path is well-trodden, documented Windows platform
capability (not something to reverse-engineer), but the keyed-mutex dead-end is exactly the kind of
thing that would cost a session to discover by trial and error — worth knowing going in that the sync
primitive has to be a query/flush pattern instead.

## 4. The matched-FOV virtual-screen formula (roadmap item 2) is a standard, three-term trig identity

For "a sane virtual screen / matched projection instead of the full-FOV stretch" — placing the
captured game frame on a flat quad in front of each eye so its *angular* size matches the game's own
rendered FOV (no stretching/squashing) rather than blindly filling the HMD's entire per-eye FOV — the
underlying math is standard and well-documented (multiple independent sources converge on the same
identity: a VR-FOV-calculation reference page, a Medium/Sensics VR-optics writeup, and general FOV
calculator references):

**`W = 2 · Z · tan(θ / 2)`** — where `Z` is the chosen distance from the eye to the virtual quad, `θ`
is the game's own horizontal FOV (already extractable from the perspective projection matrix this
project already parses per-frame for the camera override), and `W` is the quad's required width at
that distance for the projected image to subtend exactly `θ` degrees from the eye — i.e., look exactly
as wide as it did in the original flat render, just displayed in the headset instead of on a monitor.
The same formula applies per-axis for height using the vertical FOV (or derive it from `W` via the
captured frame's aspect ratio, which is simpler here since the backbuffer capture already has a fixed
aspect).

Practical notes surfaced alongside the formula:
- `Z` is a free choice (any distance works, just solve for `W` at that distance) — pick whatever reads
  comfortably; VR guidance found alongside this formula recommends against just maximizing screen size/
  FOV, since matching the *game's own* FOV (not the headset's max FOV) is what keeps the projection
  geometrically undistorted and avoids added VR discomfort from a mismatched sense of scale.
- This is the flat-quad approach specifically (not a curved-screen "virtual cinema" style projection,
  which several tools — vorpX, Bigscreen — also offer as an alternative that trades geometric
  correctness for using more of the peripheral FOV). A flat quad sized by this formula is the
  simpler, geometrically-correct option and is very likely the right first cut for this project.

## 5. Confirms AER ghosting is an industry-wide known tradeoff, not a project-specific bug to chase

Luke Ross's R.E.A.L. VR mods (AER-based flat-to-VR framework, checked as the most mature public
example of this exact technique family) offer a render-mode choice explicitly framed around this
trade-off (per the VRto3D project's community wiki, `github.com/oneup03/VRto3D/wiki/Luke-Ross-RealVR-Mods`,
since Ross's own Patreon technical posts are paywalled and weren't accessible this pass): **"AER v2"**
is described as giving "the best experience and no artifacts" but demanding "the most GPU/CPU power,"
implied to be doing meaningfully more work than naive frame-alternation (likely closer-together or
partially-duplicated per-eye rendering) specifically to suppress the ghosting that plain AER produces
— and the same wiki notes ghosting is simply accepted as unfixed in at least one supported title
(Horizon Zero Dawn). **Bottom line for FC2:** if wiggle-mode AER-to-VR shows some ghosting once the
double-buffer submit is working, that's consistent with how this whole technique family behaves
industry-wide (see item 1's root cause) — not evidence of a bug in the bridge itself. Higher-fidelity
fixes exist in principle (closer-to-true-stereo rendering) but cost meaningfully more GPU time, which
matters given this project's dev-PC-is-low-powered constraint.

## Sources (see [CREDITS.md](../CREDITS.md) for the full standing credit)

- ValveSoftware/openvr GitHub issue #1253 — SteamVR per-eye pose / alternate-eye-rendering ghosting bug report.
- Microsoft Learn — "Surface Sharing Between Windows Graphics APIs" (D3D9Ex/D3D11 interop reference).
- Microsoft Learn — `IDirect3DDevice9::GetRenderTargetData` documentation.
- GameDev.net forum threads — `GetRenderTargetData` stall/double-buffering technique; D3D9Ex/D3D11 shared-surface clarifications (community discussion, general technique, not attributed to one author).
- oneup03 / VRto3D project wiki — "Luke Ross RealVR Mods" (community-documented behavior of R.E.A.L.'s AER v2 vs legacy AER render modes).
- risa2000 — "VR headset rendered FOV calculation" reference (`risa2000.github.io/vrdocs`).
- Sensics (via Medium) — diagonal/horizontal/vertical FOV conversion reference for the matched-projection formula.
