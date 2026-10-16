# Tutorial: Scene Transitions

A trigger that moves the player from one scene to another.

You will use the **build list**, a **Trigger box**, `Scene.Load` and `Persist`.

## 1. Two scenes

1. Open the scene you have and **File > Save As...** a second one in the same project folder, for example `dungeon.scene`. Both keep the Player and floors.
2. Select nothing and open **Build scenes** in the scene settings. **Add scene** for each scene, with the hub first.

Scene numbers are positions in this list: `Scene.Load(0)` opens the first, `Scene.Load(1)` the second. Play and Export start at scene 0.

## 2. A portal in the hub

1. **Add Object > Trigger box**. Move it over the doorway and set its **Size**.
2. In its **Script** field click **New Lua** and paste:

```lua
function onTriggerEnter()
    Persist.Set("came_from", Scene.GetIndex())
    Scene.Load(1)
end
```

## 3. A portal back

Do the same in the second scene, loading scene 0:

```lua
function onTriggerEnter()
    Persist.Set("came_from", Scene.GetIndex())
    Scene.Load(0)
end
```

## 4. Read it on arrival

In either scene's script (scene settings, **Script > File**):

```lua
function onSceneCreationStart()
    local from = Persist.Get("came_from") or -1
    if from >= 0 then
        Debug.Log("Arrived from scene " .. from)
    end
end
```

## 5. Play

Press **Play**. Both scenes in the list are exported. Walk into the trigger in the first scene and you arrive in the second, and back.

## How it works

1. The player enters the box and `onTriggerEnter` fires.
2. `Persist.Set` keeps the number across the load; it holds 16 numbers and is lost on power-off.
3. `Scene.Load(i)` asks for the transition. It happens at the end of the frame, so code after the call still runs.
4. The engine loads the new scene's files and runs its scripts.

## Tips

- For a long load, set a [loading screen](../editor/scene-settings.md#loading-screen) in the scene settings. Its progress bar must be named `loading`.
- In blocks, **load scene** takes the scene number, and **when the player enters the trigger** is the event.
