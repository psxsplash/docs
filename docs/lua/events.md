# Event Callbacks

All game logic is **event-driven**. You define callback functions in your Lua scripts, and the engine calls them when events happen.

The same events drive both kinds of script. In the [block editor](../editor/block-editor.md) each event is a hat block; in Lua it is a function with the name below.

!!! tip "No game loop needed"
    You can build complex gameplay (dialogue, combat, puzzles, scene transitions) without ever using `onUpdate`. See [Patterns](patterns.md) for examples.

## Object Callbacks

Defined in the script of an object's **Script** component. All receive `self` (the object) as the first argument.

### Lifecycle

```lua
function onCreate(self)
    -- Called once after the scene finishes loading and all objects are registered.
    -- Use for initialization. Entity.Find() works here for any object.
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
function onUpdate(self, dt)
    -- Called every frame. dt is a 4.12 fixed-point delta time value.
    -- 4096 = one 30fps frame (normal speed). Higher values mean the
    -- previous frame took longer (lag compensation). Capped at 4096*4.
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
    -- Needs an Interactable component on the same object.
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

!!! note "Two players"
    These callbacks fire for presses on **either** controller and receive only the `button` id - not which player pressed it. To handle players separately, poll the [`Input.*Player1` / `Input.*Player2`](api-reference.md#input) functions directly (typically in `onUpdate`).

## Scene Callbacks

Defined in the scene's own script (**Script > File** in the [scene settings](../editor/scene-settings.md#scene-script)). No `self` argument.

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

## Agent Callbacks

Defined in the script of an object with an **Agent (NPC)** component (see [Agents](../editor/agents.md)). States are the numbers `Agent.GetState` returns.

```lua
function onStateEnter(self, newState, oldState)
    -- Called after the agent's state changes (Agent.SetState, or Idle -> Patrol on its own).
end

function onStateExit(self, state)
    -- Called just before it leaves `state`.
end

function onTargetSeen(self, target)
    -- Called when it starts seeing its target (the player unless Agent.SetTarget set another).
    -- target is nil for the player.
end

function onTargetLost(self, target)
    -- Called once the alert timeout runs out after it last saw or heard the target.
end

function onTargetReached(self)
    -- Called when it arrives at a position in the Seek, Wander or Investigate state
    -- (not while following an actor).
end

function onPatrolPoint(self, index)
    -- Called at each patrol waypoint, index counting from 0.
end

function onPathBlocked(self)
    -- Registered, but the engine never calls it yet.
end
```

## Trigger Callbacks

Defined in the script of a **Trigger box** component. **No `self` argument**: the one argument is `index`, the trigger's index in the scene.

```lua
function onTriggerEnter(index)
    -- Called when the player enters the trigger volume.
end

function onTriggerExit(index)
    -- Called when the player leaves the trigger volume.
end
```

## Callback Execution Order

On scene load, callbacks fire in this order:

1. `onSceneCreationStart()` (scene script)
2. Scene Lua file executes (top-level code runs)
3. `onCreate(self)` for all objects (after ALL objects are registered, so `Entity.Find` works)
4. `onSceneCreationEnd()` (scene script)

During gameplay, per frame:

1. Cutscenes, animations and skinned meshes tick -> `onComplete` callbacks
2. Player movement and collision -> `onCollideWithPlayer`, `onTriggerEnter` / `onTriggerExit`
3. `onUpdate(self, dt)` for each active object with the callback
4. Enable/disable events batched and fired -> `onEnable`, `onDisable`
5. Controller state updated -> `onInteract`, then `onButtonPress` / `onButtonRelease`
6. Navigation mesh update

!!! note "Collision runs before onUpdate, not after"
    A position you write in `onUpdate` is not collision-checked until the *next*
    frame.

## Event Optimization

The engine scans each script for which callbacks are defined and sets a **bitmask**. Scripts that don't define a callback never get checked for that event. This means:

- An object without `onUpdate` has **zero per-frame cost**
- An object without `onCollideWithPlayer` is never collision-checked
- Only define the callbacks you actually need
