# 2026-08-21 — First live debug: the editor API is a dead end (and that's useful)

Ran the mod against the actual game for the first time today, with a plan to
confirm the camera we'd found by reading the binary. The headline is a negative
result, but a genuinely useful one that saves us chasing the wrong thing.

## What worked

The foothold is solid. Our winmm proxy loads into Far Cry 2 every launch,
forwards Windows' audio DLL so the game is none the wiser, and runs its little
read-only probe. Across a few launches it reliably logged the engine base and
started watching the camera pointer. So Phase 1 (getting our code running inside
the game) is genuinely done.

## The thing we were testing turned out false — cleanly

We'd recovered a beautiful camera struct from the map-editor functions and hoped
the game used the same camera. It doesn't. The probe showed the editor's camera
pointer sitting at **null** the whole time — menu and in-game. I attached a
debugger to the live game to be sure, and read it directly: null. Not just the
camera — *every* editor global I checked (camera, console manager, the view
object, the scene and time managers) is null or stale junk during normal play.

The lesson: the whole `FCE_*` family of functions is the **map editor's** API.
It only comes alive when you run the Far Cry 2 editor, not when you play the
game. It was still worth disassembling — it taught us the engine's camera math
and conventions — but none of it is a live hook for gameplay.

To be thorough I grabbed a full 720 MB memory snapshot of the running game and
searched it offline for anything shaped like that editor camera (three
perpendicular unit vectors, a field-of-view value, a sensible position). Nothing
real turned up. So the game's camera is simply a *different* object with a
different layout. Good to know now.

## Two debugging gotchas (noted so we don't repeat them)

- If you launch the game *under* the debugger and then detach, the mouse stops
  working in-game (the input system gets left in a bad state). The fix is to
  launch the game normally and *attach* the debugger to it afterwards. I learned
  this the annoying way — apologies to me-of-this-afternoon.
- The engine names all its threads using a special Windows exception, so the
  debugger keeps stopping on every thread it spawns. You have to tell it to
  ignore those before the game will run smoothly under it.

## Where we go next

Straight to the reliable method we'd have reached eventually anyway: hook the
graphics API. Far Cry 2 draws through Direct3D 9, so the plan is to hook the
frame-present call, grab the real graphics device, and watch how the game hands
its view and projection matrices to the GPU. That gets us the *real* camera no
matter what class it lives in — and it's the same approach that's worked on the
other engines. Our proxy already bundles the hooking library for it; that's the
next build.

Code's in the staging repo; full technical detail in the dossier and the
dev-archive live-debug notes.
