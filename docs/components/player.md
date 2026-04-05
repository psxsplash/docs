# Player

The `PSXPlayer` component defines your game's player controller. There can only be one per scene. It controls movement, physics, camera behavior, and navigation mesh generation settings.

## Settings

### Movement

| Field | Default | Description |
|-------|---------|-------------|
| Player Height | 1.8 | Height of the collision capsule |
| Player Radius | 0.3 | Radius of the collision capsule |
| Move Speed | 0.05 | Walking speed (fixed-point units per frame) |
| Sprint Speed | 0.1 | Sprint speed |
| Jump Height | 1.0 | How high the player jumps |
| Gravity | 0.01 | Gravity strength per frame |

!!! warning "Jumping is visual only"
    Jumping currently has no collision interaction with the environment. The player moves up and down visually but does not interact with platforms, ceilings, or other geometry while airborne.

### Navigation Mesh

These settings control how the navigation mesh is generated. See [Navigation & Collision](navigation.md) for details.

| Field | Default | Description |
|---------|---------|-------------|
| Max Step Height | 0.35 | Maximum height the player can step up without jumping |
| Walkable Slope Angle | 45 | Maximum slope the player can walk on (degrees) |
| Nav Cell Size | 0.05 | Voxel resolution in XZ. Smaller = more accurate but slower export. |
| Nav Cell Height | 0.025 | Voxel resolution in Y |

!!! bug "Nav Cell Height is currently broken"
    The Nav Cell Height setting does not work correctly in the current version. Leave it at the default value.

## Camera Behavior

When navigation regions exist (walkable floor is present), the **navigation controller takes over camera movement**. The camera automatically follows the player in third-person and any manual camera changes via Lua will be overridden by the navigation controller on the next frame.

!!! note "Camera API limitations"
    The Camera Lua API (`Camera.SetPosition`, `Camera.SetRotation`) is currently not very useful in scenes with a PSXPlayer and navigation, because the nav controller continuously overrides camera state. The Camera API is primarily useful during cutscenes (which temporarily suspend the nav controller). Scenes without a PSXPlayer are largely untested.

## GTE Scaling

The **GTE Scaling** setting on the [Scene Exporter](scene-exporter.md) controls how Unity world units map to PS1 fixed-point coordinates. This is important to get right:

- **Higher values** (e.g., 200) give more precision for small details but can overflow on large scenes. Good for small, detailed environments.
- **Lower values** (e.g., 50) allow larger scenes but with less positional precision. Good for big open areas.
- **Default (100)** is a reasonable middle ground.

A good approach: if your scene is roughly 20x20 Unity units, the default 100 works well. If your scene is 100x100 units, consider lowering to 50. If it's a tiny detailed room, consider raising to 200. Watch for visual jitter or objects snapping to grid positions - that means your precision is too low.

## Gizmos

The PSXPlayer draws helpful gizmos in the Scene view:

- **Red sphere** at camera eye position
- **Green wireframe capsule** showing the collision bounds

## Physics

At runtime, the player has basic physics:

- **Gravity** pulls the player down each frame
- **Jumping** applies an upward velocity (visual only, no collision)
- **Grounding** is detected via collision with the floor
- **Frame-rate compensation** - if the game lags, physics are scaled by `dt` (4.12 fixed-point delta time, where 4096 = one 30fps frame) to maintain consistent speed
