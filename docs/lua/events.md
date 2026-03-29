# Event Callbacks

All game logic is **event-driven**. You define callback functions in your Lua scripts, and the engine calls them when events happen.

!!! tip "No game loop needed"
    You can build complex gameplay (dialogue, combat, puzzles, scene transitions) without ever using `onUpdate`. The recommended pattern is fully event-driven. See [Patterns](patterns.md) for examples.

## Object Callbacks

Defined in scripts attached to `PSXObjectExporter` components. All receive `self` (the game object) as the first argument.

### Lifecycle

```lua
function onCreate(self)
    -- Called once after the scene finishes loading and all objects are registered.
    -- Use for initialization. Entity.Find() works here for any object.
end

function onDestroy(self)
    -- Called when the object is destroyed.
end

function onEnable(self)
    -- Called when Entity.SetActive(self, true) activates the object.
end

function onDisable(self)
    -- Called when Entity.SetActive(self, false) deactivates the object.
end
```

### Per-Frame

```lua
function onUpdate(self, deltaFrames)
    -- Called every frame. deltaFrames is usually 1, but can be 2+ if the
    -- game is lagging (frame skip compensation).
    --
    -- Use sparingly. Every object with onUpdate runs this every frame.
    -- Prefer event-driven patterns instead.
end
```

### Player Interaction

```lua
function onCollideWithPlayer(self)
    -- Called when the player's collision capsule overlaps this object.
    -- Fires every frame while overlapping. Use a flag for one-time actions.
end

function onInteract(self)
    -- Called when the player presses the interact button while near this object.
    -- Requires a PSXInteractable component on the same GameObject.
end
```

### Input

```lua
function onButtonPress(self, button)
    -- Called on the frame a controller button is pressed (edge trigger).
    -- button is a number matching Input constants.
    -- Example: if button == Input.CROSS then ...
end

function onButtonRelease(self, button)
    -- Called on the frame a button is released.
end
```

## Scene Callbacks

Defined in the scene-level Lua script (attached to the Scene Exporter). No `self` argument.

```lua
function onSceneCreationStart()
    -- Called early in scene initialization, BEFORE objects are created.
    -- Good for: loading persistent data, initializing variables.
end

function onSceneCreationEnd()
    -- Called AFTER all objects are created and their onCreate() has fired.
    -- Good for: UI setup, starting ambient effects, initial game state.
end
```

## Trigger Callbacks

Defined in scripts attached to `PSXTriggerBox` components. **No `self` argument.**

```lua
function onTriggerEnter()
    -- Called when the player enters the trigger volume.
end

function onTriggerExit()
    -- Called when the player leaves the trigger volume.
end
```

## Callback Execution Order

On scene load, callbacks fire in this order:

1. `onSceneCreationStart()` (scene script)
2. Scene Lua file executes (top-level code runs)
3. `onSceneCreationEnd()` (scene script)
4. `onCreate(self)` for all objects (after ALL objects are registered, so `Entity.Find` works)

During gameplay, per frame:

1. Input state updated
2. Player movement processed
3. Interaction checks (distance, button)
4. Enable/disable events batched and fired
5. Cutscenes and animations tick
6. `onUpdate(self, deltaFrames)` for each object with the callback
7. Collision checks -> `onCollideWithPlayer`, `onTriggerEnter/Exit`

## Event Optimization

The engine scans each script for which callbacks are defined and sets a **bitmask**. Scripts that don't define a callback never get checked for that event. This means:

- An object without `onUpdate` has **zero per-frame cost**
- An object without `onCollideWithPlayer` is never collision-checked
- Only define the callbacks you actually need
