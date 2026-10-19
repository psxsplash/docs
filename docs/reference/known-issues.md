# Known Issues

What does not work yet, or works differently than you would guess. Each item was checked against the current editor and engine source.

## Scripting

### The engine never calls `onDestroy`
`Lua::OnDestroy` exists in psxsplash, but nothing calls it, and no Lua function destroys an object. Objects go away when the scene is cleared, and clearing calls no callback, so a Lua `onDestroy` never runs. Use `Entity.SetActive(object, false)` and `onDisable`.

### `Debug.DrawLine` and `Debug.DrawBox` draw nothing
Both accept their arguments and return, but neither queues a primitive. Calls are harmless. `UI.DrawLine` and `UI.DrawTriangle` do draw.

### Camera control is taken back by the player
In a scene with a Player and a nav mesh, the player controller overrides the camera every frame. Call `Camera.FollowPsxPlayer(false)` before moving the camera from Lua. Cutscenes suspend the controller by themselves.

### `Persist` holds 16 numbers
Setting more silently fails. Use the memory card API for larger or longer-lived state.

### Controller rumble has no Lua API
The motors can be driven by **Rumble** tracks in the [timeline](../editor/timeline.md#tracks), not from a script.

## Editor

### The inspector cannot set a light to runtime
Runtime point lights exist in the engine, the `Light` API and timeline tracks, but the Light component has no switch for them; set `"runtime": true` in the scene file. See [Lighting](../editor/lighting.md#runtime-lights).

### Fonts and finer nav settings live in the scene file
The UI tab picks from the scene's fonts but does not add them, and the nav bake's cell size and region merging keep their defaults unless you edit the scene file.

### No agents, sprites or networking in the editor
The engine has them (`Agent`, `Sprite`, `Net`), but the editor has no way to author them yet, so they are not in these docs.

### Scene type is stored but not used
**Exterior** and **Interior** change nothing. Culling follows the Room objects.

## Engine

### A dynamic collider does not rotate
Rotating an object from Lua does not recalculate its collision box. Position changes do.

### No CD-DA from the editor
The disc images the editor writes have one data track, and Play and Run on hardware use PCdrv. `Audio.PlayCDDA` has nothing to play. Use audio clips.

## Platform

- **Windows:** the executable is not signed. SmartScreen stops the first launch: **More info**, then **Run anyway**.
- **macOS:** Apple silicon only, and the app is not notarized. Clear the quarantine once; see [Installation](../getting-started/installation.md#macos).
- **Linux:** needs glibc 2.38 or newer.

## Reporting

Found one that is not listed? **Help > Report a Problem** opens the issue tracker.
