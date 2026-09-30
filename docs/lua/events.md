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

!!! note "Two players"
    These callbacks fire for presses on **either** controller and receive only the `button` id — not which player pressed it. To handle players separately, poll the [`Input.*Player1` / `Input.*Player2`](api-reference.md#input) functions directly (typically in `onUpdate`).

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

## Network Callbacks

Also defined in the scene-level script, and delivered once per frame from the
network layer. See [Networking](../components/networking.md).

```lua
function onNetData(payload)
    -- A reliable message from another console, sent with Net.SendData.
    -- payload is BYTES, not text: read it with string.byte.
    -- The engine never looks inside it; the format is entirely yours.
end

function onNetEvent(id, arg)
    -- A reliable numeric event, sent with Net.Send(id [, arg]).
    -- Use this when a single opcode and one number is the whole message.
end
```

!!! note "They fire only while connected"
    Both are drained before the frame's `onUpdate` calls, so state you change here
    is already correct by the time objects tick. Neither fires on the console that
    sent the message.

## Agent Callbacks

Defined in scripts attached to a GameObject that has a `PSXAgent` component. They
receive `self` like any other object callback. See [Agents](../components/agents.md)
and the [`Agent` API](api-reference.md#agent).

```lua
function onStateEnter(self, newState, oldState)
    -- The agent's state machine moved into newState. Both are numbers, 0..7.
end

function onStateExit(self, state)
    -- Fired just before onStateEnter for the state being left.
end
```

```lua
function onTargetSeen(self, target)
    -- A target came into vision range, inside the FOV cone, with line of sight.
    -- target is an ENTITY handle, or nil when the target is the player.
end

function onTargetLost(self, target)
    -- Sensing stopped, AND the alert timeout has since expired.
    -- Agent.GetLastKnownPos(actor) is where it was last seen.
end
```

!!! warning "target is nil when the target is the player"
    An agent with no explicit `Agent.SetTarget` senses the **player** by default,
    and the player has no GameObject to hand you - so `target` arrives as `nil` in
    the single most common case. Treat `nil` as "the player":

    ```lua
    function onTargetSeen(self, target)
        local who = target or Actor.GetEntity(Actor.GetPlayer())
        alert(who)
    end
    ```

!!! note "Hearing sets the alert but does not fire onTargetSeen"
    `onTargetSeen` fires only when the target is actually **seen**. A target that
    is only heard resets the alert countdown and updates the last known position
    without any callback, and can then fire `onTargetLost` when the countdown
    expires. If you need to react to sound, poll `Agent.CanHear` in `onUpdate`.

!!! note "Sensing is throttled to every third frame"
    Vision and hearing are evaluated once per three frames, so an agent notices
    something up to 100ms after it happens. The alert countdown still ticks every
    frame, so `Set Alert Timeout` means what it says.

```lua
function onTargetReached(self)
    -- An Agent.MoveTo destination was reached, within the component's
    -- Stop Distance.
end

function onPatrolPoint(self, index)
    -- A patrol waypoint was reached. index is 0-based into the waypoint list.
end
```

```lua
function onPathBlocked(self)
    -- Recognised, but never fired by the engine yet. See Known Issues.
end
```

!!! warning "There is no notification for a route that cannot be completed"
    `onPathBlocked` is wired up but the engine never raises it. An agent whose
    destination is unreachable simply stops moving with no event. Poll
    `Agent.IsMoving` against a timeout if a stuck agent would break your game.

!!! warning "target is an Entity handle, not an Actor handle"
    `Entity.*` functions work on it directly, and `Actor.*` / `Agent.*` functions
    do **not**: they return `nil`, `false` or `0` rather than raising, so the
    mistake surfaces later as a nil-index error somewhere else.

    There is no `Entity` to `Actor` conversion in the Lua API - only
    `Actor.GetEntity` going the other way. The target is usually the player, so
    `Actor.GetPlayer()` covers most cases; otherwise build a name-to-actor lookup
    once at scene start and match against `Actor.GetEntity`:

    ```lua
    local actorsByEntity = {}

    function onSceneCreationEnd()
        for i = 0, Actor.GetCount() - 1 do
            local a = Actor.FindByIndex(i)
            local e = a and Actor.GetEntity(a)
            if e then actorsByEntity[e] = a end
        end
    end
    ```

    Entity handles are stable table identities, so they work as table keys. Actor
    handles are rebuilt on every call and do not.

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

1. Network receive -> `onNetData`, `onNetEvent`
2. Cutscenes, animations and skinned meshes tick -> `onComplete` callbacks
3. Player movement and collision -> `onCollideWithPlayer`, `onTriggerEnter` / `onTriggerExit`
4. `onUpdate(self, dt)` for each active object with the callback
5. Enable/disable events batched and fired -> `onEnable`, `onDisable`
6. Controller state updated -> `onInteract`, then `onButtonPress` / `onButtonRelease`
7. Navigation mesh update
8. Agents tick -> `onStateExit` / `onStateEnter`, `onTargetSeen` / `onTargetLost`, `onTargetReached`, `onPathBlocked`, `onPatrolPoint`

!!! note "Collision runs before onUpdate, not after"
    A position you write in `onUpdate` is not collision-checked until the *next*
    frame. This is why moving an actor is done through `Agent.MoveTo` or
    `Tile.MoveActor` rather than by writing `Actor.SetPosition` directly.

## Event Optimization

The engine scans each script for which callbacks are defined and sets a **bitmask**. Scripts that don't define a callback never get checked for that event. This means:

- An object without `onUpdate` has **zero per-frame cost**
- An object without `onCollideWithPlayer` is never collision-checked
- An agent without `onTargetSeen` still senses; only the Lua call is skipped
- Only define the callbacks you actually need
