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

!!! warning "Jumping is visual only"
    Jumping currently has no collision interaction with the environment. The player moves up and down visually but does not interact with platforms, ceilings, or other geometry while airborne.

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

## GTE Scaling

The **GTE Scaling** setting on the [Scene Exporter](scene-exporter.md) controls how Unity world units map to PS1 fixed-point coordinates. This is important to get right:

- **Higher values** (e.g., 200) give more precision for small details but can overflow on large scenes. Good for small, detailed environments.
- **Lower values** (e.g., 50) allow larger scenes but with less positional precision. Good for big open areas.
- **Default (100)** is a reasonable middle ground.

A good approach: if your scene is roughly 20x20 Unity units, the default 100 works well. If your scene is 100x100 units, consider lowering to 50. If it's a tiny detailed room, consider raising to 200. Watch for visual jitter or objects snapping to grid positions - that means your precision is too low.

## Gizmos

The PSXPlayer draws helpful gizmos in the Scene view:

- **Red sphere** at camera eye position
- **Green wireframe sphere** at feet showing collision radius
- **Nav region overlay** (after building) with colored regions and portal connections

## Physics

At runtime, the player has basic physics:

- **Gravity** pulls the player down each frame
- **Jumping** applies an upward velocity (visual only, no collision)
- **Grounding** is detected via collision with the floor
- **Frame-rate compensation** - if the game lags, physics are scaled by `dt` (4.12 fixed-point delta time, where 4096 = one 30fps frame) to maintain consistent speed
