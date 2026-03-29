# Multi-Scene Projects

SplashEdit supports multiple scenes with transitions between them.

## Scene Indices

Scenes are numbered starting from **0**, in the order they appear in the Control Panel's [Scenes tab](../getting-started/control-panel.md#scenes-tab). Scene 0 is loaded first when the game starts.

!!! important
    If you reorder scenes in the Control Panel, scene indices change. Update any `Scene.Load(index)` calls in your Lua scripts accordingly.

## Loading Scenes

```lua
Scene.Load(0)    -- Load scene 0 (first scene)
Scene.Load(1)    -- Load scene 1 (second scene)

local current = Scene.GetIndex()  -- Get current scene index
```

`Scene.Load` is **deferred** - the transition happens at the end of the current frame. Code after the call still executes normally during the current frame.

## What Happens During a Scene Load

1. Current scene is cleared (all objects, UI, audio destroyed)
2. Loading screen shows (if the target scene has one assigned)
3. New `.splashpack` file is read from disc/PCdrv
4. New scene is initialized (textures uploaded, objects created, Lua scripts loaded)
5. `onSceneCreationStart` fires in the new scene
6. `onSceneCreationEnd` fires
7. All `onCreate` callbacks fire
8. Gameplay begins

## Persistent Data

Use `Persist` to carry data between scenes:

```lua
-- Save before leaving
Persist.Set("score", currentScore)
Persist.Set("health", currentHealth)
Persist.Set("came_from", Scene.GetIndex())
Scene.Load(nextScene)

-- Restore in the new scene
function onSceneCreationStart()
    local score = Persist.Get("score") or 0
    local health = Persist.Get("health") or 100
end
```

### Persist Limits

| Constraint | Limit |
|-----------|-------|
| Maximum entries | 16 key-value pairs |
| Value type | Numbers only |
| Overflow behavior | Silently fails (no error) |
| Lifetime | Survives scene loads, lost on power-off |

## Per-Scene Loading Screens

Each scene can have its own [loading screen](../components/loading-screens.md) prefab assigned on the Scene Exporter. The loading screen is a lightweight binary that loads instantly before the main scene data.

Multiple scenes can share the same loading screen prefab - the exporter deduplicates automatically.
