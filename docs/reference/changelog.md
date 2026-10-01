# Changelog

## 2.4.0 - since 2.3.0

Four new subsystems, and the Lua API to drive them. The Lua surface grows from 142
functions in 20 tables to 234 in 25, and from 13 event callbacks to 22.

### Sprites

2D sprites drawn between the 3D scene and the UI, with authored sheets and
animations. Enough to build a 2D game on psxsplash, and useful in a 3D one for
HUDs, markers and effects.

- **PSX Sprite Sheet** asset: source texture, cell size, bit depth, and a list of
  named animations over its cells
- **PSX Sprite Exporter** component, listing the sheets a scene packs
- 128-sprite pool, screen-space or bound to an actor
- Real-time animation accumulator, so a walk cycle plays at the same speed
  whatever the frame rate does
- Directional facing from a yaw, carrying frame and timer across the switch
- Per-sprite tint, flip, layer, size, and a global view offset with per-sprite
  opt-out for HUD elements

See [Sprites](../components/sprites.md) and the
[`Sprite` API](../lua/api-reference.md#sprite).

### Tilemaps

Grid-based 2D levels with per-tile walkability and painted objects.

- **PSX Tilemap** asset with an in-inspector painter
- A tileset is a sprite sheet, so tiles animate and tint like sprites
- Walkability lives on the brush, not in a parallel layer that can drift
- `Tile.MoveActor` moves an actor against the map, testing each axis separately so
  a diagonal into a wall slides instead of stopping
- `Tile.RayClear` for line of sight
- Objects painted onto the grid, with a game-defined kind byte

See [Tilemaps](../components/tilemaps.md) and the
[`Tile` API](../lua/api-reference.md#tile).

### Networking

Serial multiplayer over SIO1, for two consoles on a link cable or many through a
server.

- Symmetric handshake with host election, or slot assignment from a server
- Automatic avatar replication, interpolated between snapshots
- A reliable message channel for a game's own protocol, which the engine never
  interprets
- Host-authoritative sync for shared world objects
- Session persistence across `Scene.Load`
- Link diagnostics via `Net.Stats()`, because on a console the screen is the only
  channel there is
- Receive strategy resolves itself per target: interrupt-driven on hardware,
  polled under an emulator

The server and bridge live in their own repository,
[psnl-server](https://github.com/psxsplash/psnl_server), and are game-agnostic:
your rules attach as a plug-in.

See [Networking](../components/networking.md), the
[Networked Multiplayer tutorial](../tutorials/networking.md) and the
[`Net` API](../lua/api-reference.md#net).

### Actors

A handle for anything with a position and rotation. Actors are what sprites bind
to, what agents move, and what networking replicates.

- `Actor.Find`, `Actor.GetPlayer`, `Actor.GetEntity` to cross to an `Entity` handle
- `Actor.GetPositionXZ` returns plain integers and allocates nothing, for per-frame
  code where `GetPosition`'s four tables per call would be felt
- `Actor.GetNavRegion` and `Actor.FindPath`

See the [`Actor` API](../lua/api-reference.md#actor).

### Agents

**PSX Agent** turns a scene object into an NPC that walks the navigation mesh by
itself. Movement, sensing and patrolling are native, so an NPC that walks a route
and reacts to the player needs no per-frame Lua.

- Pathfinding across nav regions, repathed every 10 frames rather than every frame
- Vision with a range, an FOV cone and portal-graph line of sight, plus hearing as
  a plain radius
- An eight-state machine with per-state animation clips, six of the states driven
  natively and two left for your game
- Up to 8 patrol waypoints, authored in the inspector with gizmos for the cone,
  the hearing radius and the route
- Seven new Lua callbacks: `onStateEnter`, `onStateExit`, `onTargetSeen`,
  `onTargetLost`, `onTargetReached`, `onPatrolPoint`, `onPathBlocked`
- **PSX Navigation Settings**, a scene-level nav bake that does not need a
  `PSXPlayer`, so an AI-only or fully networked scene can still have regions

See [Agents](../components/agents.md) and the
[`Agent` API](../lua/api-reference.md#agent).

### Controlling something other than the player

- `Input.BindToActor(player, actor)` points a controller's built-in locomotion at
  any actor, so possession, local co-op and networked avatars all reuse the
  engine's movement rather than reimplementing it in Lua
- `Camera.SetFollowTarget` / `GetFollowTarget` / `ClearFollowTarget` do the same
  for the follow camera

### Also

- Optional per-clip leading-silence trimming on audio import, off by default
- `splashpack_dump.py` for inspecting an exported splashpack field by field

### Notes for upgrading

- **Set a Scene Network Id** on any scene that will be shared over the network.
  Left blank, the engine derives a hash from the scene's contents, which changes
  whenever you add an object - so two discs built a day apart will refuse to
  connect.
- **`Actor` handles and `Entity` handles are different.** `Entity.GetPosition` on
  an actor handle returns `nil` rather than raising, so the mistake surfaces later
  as a nil-index error somewhere unrelated. `Actor.GetEntity` crosses over
  deliberately.
