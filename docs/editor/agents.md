# Agents (NPCs)

An **agent** is an object the engine walks along the nav regions on its own: it patrols waypoints, sees and hears the player, and runs Lua events when that changes. Guards, wandering villagers and anything that chases the player are agents.

Select an object (usually one with a mesh) and choose **Add component > Agent (NPC)**. An agent object is always exported as a game object, even without a mesh, so Lua can reach it.

!!! warning "Agents need a nav bake"
    The engine only runs agents in a scene with nav regions: give the scene a **Player** or **Navigation** component and a floor with a Static collider (see [Player, Navigation & Rooms](navigation-rooms.md)). Without regions the agent stands still and senses nothing, and the inspector and the build's problems list say so.

## The component

| Field | Meaning |
|-------|---------|
| **Start enabled** | On, the agent runs from the moment the scene loads. Off, it waits for `Agent.SetEnabled(self, true)`. |
| **Move speed** | Walking speed in metres per second (stored per frame at 30 fps). Default 2. |
| **Stop distance** | How close to a waypoint or target counts as arrived, in metres. Default 0.5. |
| **Vision** | Whether it sees at all. |
| **Vision range** | How far it sees, in metres. Default 8. 0 sees nothing. |
| **Field of view** | The whole cone in degrees, centred on the way the object faces (its local +Z). 360 sees all around. Default 90. |
| **Region hops** | How many nav regions away the target may be and still be seen: 0 is the agent's own region, at most 8. Default 2. |
| **Hearing range** | How far it hears, in metres. 0 (the default) hears nothing. |
| **Alert timeout** | Frames after it last sensed the target before `onTargetLost` runs. Default 90. |
| **Waypoints** | Patrol points in world metres, walked in order and back to the first. **At object** adds one where the object stands, **At view centre** one under the centre of the viewport. The arrows reorder them; the bin deletes one. The engine keeps 8. |
| **Idle ... Custom** | With a [Skinned animation](skinned-animation.md) component on the same object: the clip the agent plays when it enters each state. Without one the section says so. |
| **Export** | What the agent adds to the scene file: 28 bytes plus 12 per waypoint. |

The engine measures ranges as the larger of the X and Z distances, so a range is a **square** around the agent, not a circle. With the agent selected the viewport draws the vision cone cut by that square in pink, the hearing square in blue, and the numbered waypoints joined in patrol order. Drag a waypoint's dot to move it across the ground at its own height. The overlay toggle in the viewport toolbar hides them.

The inspector (and the build's problems list) warns about more than 8 waypoints (the rest are left out), a field of view outside 1 to 360 degrees, a state clip on an object without a skin or past its clip list, and a scene with no nav regions.

## States

An agent is always in one of eight states. `Agent.GetState` returns the number, `Agent.SetState` changes it.

| # | State | What the engine does in it |
|---|-------|---------------------------|
| 0 | Idle | Switches to Patrol when patrolling is on and there are waypoints. |
| 1 | Patrol | Walks to the current waypoint, then the next (`onPatrolPoint` at each). |
| 2 | Seek | Follows its target actor. |
| 3 | Flee | Moves away from its target actor. |
| 4 | Attack | Nothing: Lua drives it. |
| 5 | Wander | Walks to random spots around its nav region. |
| 6 | Investigate | Walks to where it last saw or heard the target. |
| 7 | Custom | Nothing: Lua drives it. |

Seeing the player does not change the state by itself: switch states from the events below, for example Seek in `onTargetSeen` and Investigate in `onTargetLost`.

## Lua

The `Agent.*` functions take an **actor** handle: `self` in the agent's own script, or `Actor.Find("Guard")` anywhere. Positions are in engine units, Y down, like `Actor.GetPosition`: the scene's metres divided by its GTE scaling (100 by default), so a waypoint at (-4, 0, -4.5) m reads back as -0.040, 0, -0.045. See the [`Agent` API](../lua/api-reference.md#agent).

```lua
-- On the Guard's own script.
function onTargetSeen(self, target)
    Agent.SetState(self, 2)            -- Seek
    Agent.SetTarget(self, Actor.GetPlayer())
end

function onTargetLost(self, target)
    Agent.SetState(self, 6)            -- Investigate the last known position
end

function onTargetReached(self)
    Agent.SetState(self, 1)            -- back to the patrol
end

function onPatrolPoint(self, index)
    Debug.Log("reached waypoint " .. index)
end
```

The agent events are object callbacks on the agent's own script (see [Events](../lua/events.md#agent-callbacks)): `onStateEnter(self, newState, oldState)`, `onStateExit(self, state)`, `onTargetSeen(self, target)`, `onTargetLost(self, target)` (target is nil when it is the player), `onTargetReached(self)` (arrived at a position in Seek, Wander or Investigate; not while following an actor), `onPatrolPoint(self, index)` (0-based) and `onPathBlocked(self)`. The engine registers `onPathBlocked` but never calls it yet.
