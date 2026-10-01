# Agents

An **agent** is a scene object that walks the navigation mesh by itself. Add a
`PSXAgent` component and the engine handles pathfinding, movement, vision, hearing
and a small state machine natively. An NPC that patrols a route and reacts to the
player needs no per-frame Lua at all.

Agents are built on [navigation regions](navigation.md), so a scene needs a baked
nav mesh before they can move. That comes from a `PSXPlayer`, or - in a scene with
no player at all - from a
[`PSXNavigationSettings`](navigation.md#psxnavigationsettings) component. The agent
inspector tells you which one it found, and errors if there is neither.

## Adding an agent

Add **PSX Splash > PSX Agent** to a GameObject that already has a
`PSXObjectExporter`. The component requires one and will add it for you.

Give the GameObject a name you can look up from Lua:

```lua
local guard = Actor.Find("guard_1")
Agent.SetState(guard, 1)   -- Patrol
```

## Settings

### Movement

| Field | Default | Description |
|-------|---------|-------------|
| Start Enabled | on | Whether the agent ticks from scene load. Off means it stands still until `Agent.SetEnabled`. |
| Move Speed | 3.0 | World units per second, the same scale as the player's Move Speed |
| Stop Distance | 0.2 | How close counts as arrived. Reaching it fires `onTargetReached`. |

### Vision

| Field | Default | Description |
|-------|---------|-------------|
| Has Vision | off | Enables the vision sense |
| Vision Range | 8 | Maximum sight distance in world units. 0 disables vision whatever the checkbox says. |
| Vision Fov Degrees | 90 | The **full** cone. 360 is omnidirectional inside the range sphere. |
| Vision Region Depth | 2 | How many nav-region portal hops line of sight may cross |

**Vision Region Depth** is the cost dial. `0` means the agent can only see inside
its own region, which is cheap and enough for a corridor. Each extra hop lets sight
pass through one more doorway and costs more CPU per check. 2 to 3 covers most
interiors.

### Hearing

| Field | Default | Description |
|-------|---------|-------------|
| Has Hearing | off | Enables the hearing sense |
| Hearing Range | 6 | Radius in world units |

Hearing is **range only**. No geometry is consulted, so a target behind a wall is
still heard. That is deliberate: it is what makes sound useful as the sense that
gets around cover.

### Alert

| Field | Default | Description |
|-------|---------|-------------|
| Alert Timeout Frames | 60 | Frames at 30fps the agent stays alert after losing the target |

The timeout is what stops an agent flickering between alerted and calm as the
target steps in and out of a doorway. `onTargetLost` fires when it expires, not
when sensing stops.

### Patrol Waypoints

A world-space list, visited in order and looped. **Maximum 8**; the inspector
truncates anything longer. Reaching one fires `onPatrolPoint(self, index)` with a
0-based index.

An agent in the Idle state with waypoints moves to Patrol on its own.

### State Animations

One [animation clip](animations.md) per state. Entering a state plays its clip
automatically; leave a slot empty to keep the previous animation running.

## The state machine

| Value | State | What the engine does |
|---|---|---|
| 0 | Idle | nothing, but moves to Patrol if waypoints exist |
| 1 | Patrol | walks the waypoint list, looping |
| 2 | Seek | paths toward the current target actor |
| 3 | Flee | moves away from the current target actor |
| 4 | Attack | **nothing** - yours to drive |
| 5 | Wander | picks a random point near its nav region every ~60 frames |
| 6 | Investigate | walks to the last known target position |
| 7 | Custom | **nothing**, and never entered on its own |

Idle, Patrol, Seek, Flee, Wander and Investigate are driven natively. Attack and
Custom exist so your Lua has somewhere to put behaviour the engine should stay out
of.

Transitions between them are yours: sensing raises `onTargetSeen`, and it is your
script that decides whether that means Seek, Flee or Attack.

```lua
function onTargetSeen(self)
    local me = Actor.Find(...)
    Agent.SetState(me, 2)   -- Seek
end

function onTargetLost(self)
    local me = Actor.Find(...)
    Agent.SetState(me, 6)   -- Investigate the last known position
end
```

## Editor gizmos

Selecting an agent draws its vision cone in yellow, its hearing radius in cyan, and
its patrol route in green. Agents with any of the three configured draw a dimmer
version even when not selected, so you can see a level's coverage at a glance.

## Costs

- Vision and hearing are evaluated **once per three frames**, so an agent notices
  something up to 100ms late. The alert countdown ticks every frame regardless.
- Paths are recomputed every **10 frames** while moving, not every frame.
- Each agent exports as a 28-byte record, so the memory cost is in the navigation
  mesh rather than in the agents themselves.

!!! tip "Vision Region Depth is the first thing to turn down"
    Line-of-sight across the portal graph is the expensive part of an agent. If a
    scene with several agents is dropping frames, drop the depth to 1 before
    changing anything else.

## Pitfalls

!!! warning "Agent functions take actor handles, callbacks give entity handles"
    `Agent.*` and `Actor.*` take a handle from `Actor.Find`. The `target` argument
    of `onTargetSeen` is an **entity** handle, and there is no conversion back.
    See [Event Callbacks](../lua/events.md#agent-callbacks) for the lookup pattern.

!!! warning "Agent.SetVisionAngle takes the half angle"
    The inspector's **Vision Fov Degrees** is the full cone; the Lua setter takes
    half of it. The 90 degree cone you authored is `Agent.SetVisionAngle(a, 45)`.

!!! warning "An unreachable destination is silent"
    `onPathBlocked` exists but the engine never fires it. An agent that cannot
    complete a route simply stops. Poll `Agent.IsMoving` against a timeout if that
    would break your game.

## See also

- [Navigation & Collision](navigation.md) - agents need a baked nav mesh
- [Animations](animations.md) - the clips bound to each state
- [`Agent` API reference](../lua/api-reference.md#agent)
- [Agent callbacks](../lua/events.md#agent-callbacks)
