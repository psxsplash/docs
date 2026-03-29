# Navigation & Collision

SplashEdit uses [DotRecast](https://github.com/ikpil/DotRecast) (a C# port of Recast Navigation) to generate navigation meshes for your scene. The nav mesh tells the PS1 runtime where the player can walk.

## How Navigation Works

1. SplashEdit collects geometry from all `PSXObjectExporter` components
2. It voxelizes the scene into a 3D grid at the configured cell size
3. Walkable surfaces are identified based on slope angle and step height
4. Convex navigation regions are generated (polygons with up to 6 vertices each)
5. Portals (connections between adjacent regions) are computed with height deltas
6. The result is exported as binary data in the splashpack

## Navigation Settings

All navigation settings live on the `PSXPlayer` component:

| Setting | Default | Description |
|---------|---------|-------------|
| Nav Cell Size | 0.05 | XZ voxel resolution. Smaller = more accurate edges, slower export, more memory. |
| Nav Cell Height | 0.025 | Y voxel resolution. |
| Max Step Height | 0.35 | Maximum height difference the player can step over. |
| Walkable Slope Angle | 45 | Maximum slope the player can walk on, in degrees. |

!!! bug "Nav Cell Height is broken"
    The Nav Cell Height setting does not work correctly in the current version. Leave it at the default.

!!! warning "Step height must match your geometry"
    If your stairs are 0.5 units tall but Max Step Height is 0.35, the player won't be able to climb them. The voxelization is correct but `walkableClimb` prevents connections between height levels.

## Collision Types

Set on each `PSXObjectExporter`:

| Type | Description |
|------|-------------|
| None | No collision detection. Walkability comes from the nav mesh. |
| Static | Solid wall. Player is blocked. Use for walls, pillars, fixed obstacles. |
| Dynamic | Moveable collider. For objects repositioned via Lua at runtime. |

!!! warning "Dynamic collision AABB limitation"
    Rotating objects via Lua does **not** update their AABB collision bounds. The collision box stays in the original orientation. Only position changes work correctly for dynamic colliders.

## How Collision Works at Runtime

The PS1 runtime uses AABB (axis-aligned bounding box) collision:

- **Static colliders** block player movement
- **Dynamic colliders** are checked against the player each frame
- **Trigger boxes** fire Lua events on enter/exit (see [Trigger Boxes](trigger-boxes.md))
- **Line-of-sight checks** are used by [Interactables](interactables.md) when `requireLineOfSight` is enabled

## Tips

!!! tip "Cell size trade-off"
    Smaller cell sizes produce more accurate navigation edges but generate more regions and use more memory. For most scenes, the defaults work well.
