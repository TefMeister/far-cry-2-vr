# Leads for both open Far Cry 2 rows, from phunkaeg's VR Modding Playbook

Source: https://github.com/phunkaeg/vr-modding-playbook (code MIT, prose CC BY 4.0), snapshot
read 2026-09-23 (last commit 2026-09-20). Credit phunkaeg if anything here is used.

All claims below are `[reported]`: they are the playbook author's records, not re-checked by us.

## Their own Far Cry 2 project is further along than ours

`sources.yml` → `FarCry2-vr` (in-house, unreleased): Dunia, D3D10 (D3D9 selectable), x86, marked
stereo HEADSET-accepted, hands HEADSET-accepted. They also studied a second one,
**MonsterDeadWood FC2VR** (D3D9, native injector, same-frame stereo, working motion-controller hands).

## Row: "nothing behind you is drawn" (head turns the picture, not the engine camera)

This is a culling problem, and both projects above documented the culling data:

- MonsterDeadWood FC2VR: exact-build culling sphere centre/radius layout and the culling seam,
  documented against the **GOG** `Dunia.dll` (SHA256 7B82...). Side planes 2-5 versus depth planes
  0-1 support a "side-only binocular union" widening. Their Uplay/Steam bytes differ, so the
  addresses are leads only, not offsets to paste.
- Their rule from it: observe the culling call first, change nothing, before forcing anything; an
  intrusive force-pass changed the call count and spoiled the measurement.
- Chapter 17 (`docs/17-teardown-fc2vr-native-stereo.md`) §"FC2VR: four failures" R1-R4: hold the
  engine camera as a restorable transaction; the engine keeps observing while you hold its camera;
  one flag serving two meanings will flap; publish the eye pair, never half of one.

## Row: "two guns at a real IPD" (weapon gets the wrong separation)

`docs/failure-atlas.md` FAIL-STR-032: *the world converges but the weapon does not* — cheap test:
park the hand, change the world FOV, see whether the weapon holds its size. Cause: the weapon /
near pass has its own projection that inherits, or is left out of, the world's eye setup. Recipe
STR-011 in `docs/pattern-catalog.md`. Also FAIL-HAND-037 (constant left/right offset on the hands
that does not change with distance).

## Hands, when we get there

Right-palm bone 80 carries the weapon; bones 80..95 must be driven together (palm-only detached
the wrist and fingers); the write must happen inside the animator evaluation, because a write from
the camera/render hook is overwritten by skeleton evaluation.
