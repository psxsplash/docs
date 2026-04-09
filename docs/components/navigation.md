# Navigation & Collision

SplashEdit uses [DotRecast](https://github.com/ikpil/DotRecast) (a C# port of Recast Navigation) to generate navigation meshes for your scene. The nav mesh tells the PS1 runtime where the player can walk.

## How Navigation Works

1. SplashEdit collects geometry from all `PSXObjectExporter` components
2. It voxelizes the scene into a 3D grid at the configured cell size
3. Walkable surfaces are identified based on slope angle and step height
4. The voxelized regions are partitioned using the selected algorithm (Watershed, Monotone, or Layer)
5. Convex navigation regions are generated (polygons with up to 8 vertices each)
6. A detail mesh is generated to accurately sample terrain height within each region
7. A least-squares floor plane is fit to each region using both polygon corner and detail mesh interior vertices
8. Portals (connections between adjacent regions) are computed with height deltas
9. The result is exported as binary data in the splashpack

## Navigation Settings

All navigation settings are configured directly on the `PSXPlayer` component inspector. The nav builder UI is integrated into the player inspector — there is no separate Nav Region Builder window.

### Core Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Nav Cell Size | 0.05 | XZ voxel resolution. Smaller = more accurate edges, slower export, more memory. |
| Nav Cell Height | 0.025 | Y voxel resolution. Affects how finely height differences are resolved. |
| Max Step Height | 0.35 | Maximum height difference the player can step over. |
| Walkable Slope Angle | 46 | Maximum slope the player can walk on, in degrees. |

### Advanced Settings

These settings are in a foldout section on the PSXPlayer inspector and give fine control over region generation:

| Setting | Default | Description |
|---------|---------|-------------|
| Partition Method | Watershed | Algorithm for dividing walkable space into regions (see below). |
| Min Region Area | 8 | Regions smaller than this (in voxels) are removed entirely. Raise to eliminate tiny slivers. |
| Merge Region Area | 20 | Regions smaller than this get merged into their largest neighbor. Lower = more regions with better terrain accuracy. Higher = fewer regions, less PS1 memory. |
| Max Simplify Error | 1.3 | Maximum deviation when simplifying contours (world units). Lower = edges follow voxel boundaries more closely. |
| Max Edge Length | 12.0 | Maximum polygon edge length (world units). Shorter = smaller regions with better height fit on terrain. |
| Detail Sample Distance | 6.0 | Detail mesh height sampling distance as a multiplier of Cell Size. Lower = more height samples for better floor planes. |
| Detail Max Error | 0.025 | Maximum height error for detail mesh generation (world units). Lower = more accurate floor Y on slopes. |
| Max Plane Error | 0.15 | Warning threshold for floor plane fit error (world units). Regions exceeding this are flagged in the build statistics. |

### Partition Methods

| Method | Best For | Description |
|--------|----------|-------------|
| **Watershed** | Indoor / architectural | Produces clean rectangular regions following wall edges. Can create oversized regions on undulating terrain. |
| **Monotone** | Open terrain / natural | Produces more uniform, predictable regions that follow terrain contours. Better for hills and valleys. |
| **Layer** | Multi-level / overlapping | Keeps height layers separated properly for overlapping floors, bridges, and multi-story buildings. |

### Presets

The PSXPlayer inspector provides three preset buttons that configure all advanced parameters at once:

| Preset | Cell Size | Partition | Merge Area | Simplify Error | Description |
|--------|-----------|-----------|------------|----------------|-------------|
| **Indoor** | 0.05 | Watershed | 20 | 1.3 | Clean regions for architectural geometry |
| **Terrain** | 0.08 | Monotone | 6 | 0.4 | Smaller regions that track natural terrain better |
| **Multi-Level** | 0.05 | Layer | 12 | 0.8 | Proper floor separation for overlapping levels |

!!! tip "Start with a preset"
    Choose the preset closest to your scene type, then fine-tune individual parameters as needed.

!!! warning "Step height must match your geometry"
    If your stairs are 0.5 units tall but Max Step Height is 0.35, the player won't be able to climb them. The voxelization is correct but `walkableClimb` prevents connections between height levels.

## Building & Previewing

The PSXPlayer inspector has a **Build Nav Regions** button that runs the DotRecast pipeline without a full scene export. After building:

- **Statistics** are shown including region count, portal count, total binary size, and per-region plane error
- **Scene view gizmos** display the nav mesh with colored regions
- Regions with high plane error are tinted **red** as a visual warning
- Portal connections are shown as yellow lines between regions

Use **Clear** to remove the preview data. The full export pipeline also builds nav regions automatically.

!!! tip "PS1 budget warnings"
    The statistics panel warns when region counts exceed **128** (caution) or **256** (critical), and when the binary size exceeds **8KB** (caution) or **16KB** (critical). These thresholds help you stay within PS1 memory constraints.

## Height-Aware Region Selection

The PS1 runtime uses a height-aware approach when finding which navigation region the player is standing on. When multiple regions overlap at the same XZ position (e.g., a floor plus an elevated walkway above it), the runtime selects the region whose floor plane is closest to the player's current Y position. This prevents the player from "snapping" to the wrong floor in multi-level environments.

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
    Smaller cell sizes produce more accurate navigation edges but generate more regions and use more memory. Start with a preset and adjust from there.

!!! tip "Terrain scenes"
    For natural terrain with hills and valleys, use the **Terrain** preset or **Monotone** partitioning. Lower the **Merge Region Area** to prevent regions from spanning across valleys, which causes inaccurate floor planes.

!!! tip "Monitor plane error"
    After building, check the per-region statistics. Regions with high plane error will have incorrect floor Y values at runtime, causing the player to float or sink. Reduce **Merge Region Area** and **Max Edge Length** to break large regions into smaller ones that fit the terrain better.
