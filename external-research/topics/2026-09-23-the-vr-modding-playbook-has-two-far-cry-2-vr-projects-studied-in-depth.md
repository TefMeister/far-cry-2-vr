# The VR Modding Playbook has two Far Cry 2 VR projects studied in depth

**Found:** 2026-09-23, `/gr` estate sweep. Tefa pointed at the playbook the same morning; the modding
side already dropped its row-by-row leads into `engine-research/inbox/`
(`2026-09-23-mod-phunkaeg-playbook-fc2-culling-and-weapon-leads.md`). This topic indexes the source in
this lane and adds what that drop did not cover.
**Source:** phunkaeg, *VR Modding Playbook* — <https://github.com/phunkaeg/vr-modding-playbook> (code
MIT, prose CC BY 4.0, last commit 2026-09-20). Chapter
[17 — FC2VR native stereo](https://github.com/phunkaeg/vr-modding-playbook/blob/main/docs/17-teardown-fc2vr-native-stereo.md).
Read on GitHub; nothing copied.

## What is in it for this game `[reported]`

- **Two Far Cry 2 VR projects:** the author's own unreleased *FarCry2-vr* (Dunia, D3D10 with D3D9
  selectable, stereo and hands accepted in a headset) and *MonsterDeadWood FC2VR* (D3D9, native
  injector, same-frame stereo, working controller hands). The playbook marks FC2 as reaching the
  **top rung** of its stereo ladder: the engine's own frame-graph and world-execute steps are
  reachable as vtable slots, so the world can be rendered twice by the engine itself.
- **Four shipped failures (R1–R4)** from FC2VR, each generalised: hold the engine camera as a
  restorable transaction that fails visibly; the engine keeps watching while you hold its camera; one
  flag serving two meanings will flap; publish the eye pair, never half of one.
- **Culling data** for the "nothing behind you is drawn" row, and **FAIL-STR-032 / STR-011** for the
  "two guns at a real IPD" row: covered in the modding drop above, not repeated here.

## What the modding drop did not cover

1. **Camera delivery decides how much of R1–R4 you pay for.** The chapter's table puts Far Cry 2 in
   the worst column: **the camera reaches the renderer as a global you overwrite**, so the
   borrow-restore-freeze machinery is unavoidable. Where an engine passes the camera as a parameter
   (CryEngine, id Tech 2), that whole bug family is designed out `[reported]`. Useful to state
   plainly in our dossier, because it predicts which bugs to expect.
2. **Store builds, proven rather than assumed.** GOG, Steam and Ubisoft Connect ship different
   `Dunia.dll` files. FC2VR identifies the build by SHA256 and carries a verified offset table per
   build. It proved **Steam and Ubisoft are the same port** by matching every section hash and the
   embedded debug path
   (`d:\dev\fc2relaunch\fcx-branches\fc2-pc-uplay\bin\Dunia.pdb`) `[reported]`. Two cheap tricks for
   us: the embedded PDB path is a build-identity string, and every anchor should carry a validator
   that re-derives it from the file before anything runs.
3. **MonsterDeadWood's culling addresses are for the GOG build.** Our copy is Steam, so they are
   leads to re-find, never offsets to use `[reported]`.

## Transfers

- **Far Cry 3 Blood Dragon** is also Dunia. The store-build method and the "camera is a global"
  expectation are the first things to check there `[hypothesis]`.

## Credits

phunkaeg (*VR Modding Playbook*, FarCry2-vr); MonsterDeadWood (FC2VR), as studied in the playbook.
