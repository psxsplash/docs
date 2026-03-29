# Tutorial: Scene Transitions

Create portals that transport the player between different scenes.

## What You'll Learn

- Multi-scene project setup
- `Scene.Load` for scene transitions
- `Persist` for carrying data across scenes
- Trigger boxes as portal zones

## Step 1: Create Two Scenes

1. Create **Scene A** (e.g., "Hub") with a Scene Exporter, Player, and floor
2. Create **Scene B** (e.g., "Dungeon") with a Scene Exporter, Player, and floor
3. In the Control Panel **Scenes** tab, add both scenes:
    - Scene A at index 0
    - Scene B at index 1

!!! important
    Scene indices are determined by position in the Scenes list. `Scene.Load(0)` loads the first scene, `Scene.Load(1)` loads the second.

## Step 2: Portal in Scene A

1. Create an empty GameObject at the portal location
2. Add `PSXTriggerBox`, set size to cover the doorway/portal area

Create `portal_to_dungeon.lua`:

```lua
function onTriggerEnter()
    setStatus("Entering the dungeon...")
    Persist.Set("came_from", Scene.GetIndex())
    Scene.Load(1)  -- Load Scene B
end
```

Assign this script to the trigger box.

## Step 3: Portal in Scene B

Same setup with `portal_to_hub.lua`:

```lua
function onTriggerEnter()
    setStatus("Returning to hub...")
    Persist.Set("came_from", Scene.GetIndex())
    Scene.Load(0)  -- Load Scene A
end
```

## Step 4: Restore Data in Each Scene

In each scene's `scene.lua`:

```lua
function onSceneCreationStart()
    local prevScene = Persist.Get("came_from") or -1
    if prevScene >= 0 then
        Debug.Log("Arrived from scene " .. prevScene)
    end

    -- Restore score
    local score = Persist.Get("score") or 0
    Debug.Log("Score: " .. score)
end
```

## Step 5: Build and Test

Build with both scenes in the list. Walk into the portal in Scene A to go to Scene B. Walk into the portal in Scene B to return.

## How It Works

1. Player walks into the trigger box -> `onTriggerEnter` fires
2. `Persist.Set` saves the current scene index (so the target scene knows where we came from)
3. `Scene.Load(index)` requests a transition (deferred to end-of-frame)
4. The runtime blanks the screen, shows the loading screen (if any), loads the new splashpack
5. The new scene's Lua scripts execute, can read `Persist.Get` for carried data

## Tips

!!! tip "Scene.Load is deferred"
    Code after `Scene.Load()` still executes during the current frame. The transition happens at the end of the frame. Don't do cleanup after `Scene.Load` - it's not needed.

!!! tip "Loading screens"
    For longer scenes, assign a [Loading Screen](../components/loading-screens.md) prefab on the Scene Exporter. This shows while the scene data is being loaded.

!!! tip "Persist limits"
    Only 16 key-value pairs available. Plan your persistent state carefully. Use meaningful key names.
