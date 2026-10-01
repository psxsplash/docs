# Player

The `PSXPlayer` component defines your game's player controller. There can only be one per scene. It controls movement, physics, camera behavior, and navigation mesh generation settings.

## Settings

### Movement

| Field | Default | Description |
|-------|---------|-------------|
| Player Height | 1.8 | Height of the collision capsule |
| Player Radius | 0.5 | Radius of the collision capsule |
| Move Speed | 3.0 | Walking speed (world units per second) |
| Sprint Speed | 8.0 | Sprint speed (world units per second) |

!!! note "Jumping and platforms"
    Jumping works with navigation regions. The player can jump onto and off of platform regions and walkoff edges. When airborne, the player falls under gravity and lands on whichever region they touch. See [Navigation - Platforms](navigation.md#platforms) for how to set up walkable edges.

### Navigation Mesh

All navigation mesh generation settings are configured directly on the PSXPlayer inspector. The nav builder UI is integrated here — there is no separate Nav Region Builder window. See [Navigation & Collision](navigation.md) for full details.

**Core settings:**

| Field | Default | Description |
|---------|---------|-------------|
| Max Step Height | 0.35 | Maximum height the player can step up without jumping |
| Walkable Slope Angle | 46 | Maximum slope the player can walk on (degrees) |
| Nav Cell Size | 0.05 | Voxel resolution in XZ. Smaller = more accurate but slower export. |
| Nav Cell Height | 0.025 | Voxel resolution in Y |

**Advanced settings** (in a foldout):

| Field | Default | Description |
|---------|---------|-------------|
| Partition Method | Watershed | Region partitioning algorithm (Watershed, Monotone, or Layer) |
| Min Region Area | 8 | Minimum region size in voxels. Smaller regions are removed. |
| Merge Region Area | 20 | Threshold below which regions merge into neighbors. Lower = more regions, better terrain accuracy. |
| Max Simplify Error | 1.3 | Contour simplification tolerance (world units) |
| Max Edge Length | 12.0 | Maximum polygon edge length (world units) |
| Detail Sample Dist | 6.0 | Detail mesh height sampling density |
| Detail Max Error | 0.025 | Maximum detail mesh height error (world units) |
| Max Plane Error | 0.15 | Floor plane fit warning threshold (world units) |

Three **presets** are available for common scene types: **Indoor**, **Terrain**, and **Multi-Level**. See [Navigation — Presets](navigation.md#presets) for details.

The inspector also provides **Build**, **Clear**, and **Preview** controls for testing navigation without a full export.

### Jump & Gravity

| Field | Default | Description |
|-------|---------|-------------|
| Jump Height | 2.0 | Peak jump height in world units |
| Gravity | 20.0 | Downward acceleration in world units per second squared |

## Camera Behavior

When navigation regions exist (walkable floor is present), the **navigation controller takes over camera movement**. The camera automatically follows the player in third-person and any manual camera changes via Lua will be overridden by the navigation controller on the next frame.

!!! note "Camera API limitations"
    The Camera Lua API (`Camera.SetPosition`, `Camera.SetRotation`) is currently not very useful in scenes with a PSXPlayer and navigation, because the nav controller continuously overrides camera state. Use `Camera.FollowPsxPlayer(false)` to take manual control of the camera from Lua. The Camera API is also usable during cutscenes, which temporarily suspend the nav controller.

### Attaching a camera (or anything else) to a moving entity

!!! important "Unity's parent/child hierarchy is not carried to the runtime"
    SplashEdit exports each object's transform, not the Unity scene hierarchy.
    If you parent a camera or prop to a moving object in the Unity editor for
    convenience, that parenting relationship does **not** exist at runtime —
    the exported objects are flat. To attach something to a moving entity at
    runtime, use `Entity.SetParent` from Lua.

`Entity.SetParent(parent, child, offset)` snaps `child` to `parent`: the child is
placed at the parent's position plus `offset` (a `Vec3` in the **parent's local
space**) and its rotation is set to match the parent's. It is a one-shot snap,
not a persistent link — call it every frame (typically from `onUpdate`) to keep
the child attached as the parent moves or turns. See
[`Entity.SetParent`](../lua/api-reference.md#parenting) for the full reference.

```lua
-- Keep a camera-mount object riding just above a moving platform
function onUpdate(self, dt)
    local platform = Entity.Find("Platform")
    Entity.SetParent(platform, self, Vec3.new(FixedPoint.new(0), FixedPoint.new(1.5), FixedPoint.new(0)))
end
```

## GTE Scaling

The **GTE Scaling** setting on the [Scene Exporter](scene-exporter.md) controls how Unity world units map to PS1 fixed-point coordinates. The exporter divides every Unity coordinate by it, so with the default 100, 100 Unity units become 1.0 on the PlayStation, stored in 4.12 fixed point (steps of 1/4096).

- **Higher values** shrink the scene on the PlayStation side: more room for big scenes, and coarser positions. Good for large open areas.
- **Lower values** give finer positions for small, detailed rooms, at the cost of range.
- **Default (100)** is a reasonable middle ground.

Watch for visual jitter or objects snapping to grid positions - that means GTE Scaling is too high for the detail you have. Objects stretching, wrapping or vanishing far from the origin mean it is too low.

**Draw distance follows from it.** Geometry more than 4 PS1 units from the camera is not drawn: the ordering table has 16384 slots at 1/4096 of a unit each. Multiply by GTE Scaling for Unity units, so the default 100 gives a 400-unit draw distance and 200 gives 800. If distant scenery disappears, raise GTE Scaling.

## Gizmos

The PSXPlayer draws helpful gizmos in the Scene view:

- **Red sphere** at camera eye position
- **Green wireframe sphere** at feet showing collision radius
- **Nav region overlay** (after building) with colored regions and portal connections

## Physics

At runtime, the player has basic physics:

- **Gravity** pulls the player down each frame
- **Jumping** applies an upward velocity. The player can land on platform regions and walk off edges.
- **Grounding** is detected when the player's Y position reaches the floor of a nav region
- **Coyote time** allows the player to still jump shortly after walking off an edge
- **Velocity cap** limits the maximum downward speed during long falls
- **Frame-rate compensation** - if the game lags, physics are scaled by `dt` (4.12 fixed-point delta time, where 4096 = one 30fps frame) to maintain consistent speed
