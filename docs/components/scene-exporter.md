# Scene Exporter

Every scene needs exactly one `PSXSceneExporter` component on a root GameObject. This is the master controller for the export pipeline.

## Adding a Scene Exporter

Go to **GameObject -> PlayStation 1 -> Scene Exporter**. SplashEdit enforces a singleton - only one per scene.

## Settings

| Field | Description | Default |
|-------|-------------|---------|
| Scene Type | **Interior** or **Exterior** (see below) | Exterior |
| GTE Scaling | Vertex position scale factor. Higher = more precision, but can overflow on large scenes. | 100.0 |
| Scene Lua File | Optional [Lua script](../lua/index.md) that runs at the scene level. Used for initialization, UI setup, and shared functions. | None |
| Fog Color | RGB color for distance fog | Black |
| Fog Enabled | Toggle fog on/off | Off |
| Fog Density | How quickly fog appears with distance | 0 |
| Cutscenes | Array of [PSXCutsceneClip](cutscenes.md) assets used in this scene | Empty |
| Animations | Array of [PSXAnimationClip](animations.md) assets used in this scene | Empty |
| Loading Screen Prefab | Optional prefab shown while the scene [loads](loading-screens.md) | None |

## Scene Types

### Exterior

Uses a **Bounding Volume Hierarchy (BVH)** for frustum culling. The BVH groups triangles spatially and culls entire branches that fall outside the camera's view.

Best for: open areas, outdoor environments.

!!! warning
    Exterior BVH rendering currently has significant issues and needs a full rewrite. See [Known Issues](../reference/known-issues.md). **Use Interior scene type whenever possible.**

### Interior

Uses a **room/portal occlusion system**. Only rooms visible through portal connections are rendered. Much more efficient for enclosed spaces.

Best for: buildings, dungeons, hallways, any enclosed space.

!!! important
    Interior scenes **must have at least one [PSXRoom](rooms-portals.md) defined**. Without rooms, rendering glitches will occur.

## Debug Visualization

The inspector has debug toggles at the bottom:

| Toggle | Description |
|--------|-------------|
| Preview BVH | Show the bounding volume hierarchy in the Scene view |
| Preview Rooms/Portals | Show room volumes (green) and portal connections (orange) |
| BVH Preview Depth | How deep into the BVH tree to visualize |

!!! bug "BVH Preview is broken"
    The BVH preview visualization is not working correctly in the current version. The Rooms/Portals preview works fine.

## Export Pipeline

When you export (via the [Control Panel](../getting-started/control-panel.md) or directly), the exporter runs this sequence:

1. Collects all `PSXObjectExporter` components in the scene
2. Quantizes and deduplicates textures, packs into VRAM
3. Converts meshes to PS1 format with pre-baked lighting
4. Builds BVH (exterior) or room/portal structure (interior)
5. Generates navigation regions from walkable geometry
6. Exports UI canvases, custom fonts, and font pixel data
7. Exports cutscenes, animations, audio, interactables, and trigger boxes
8. Writes the complete `.splashpack` binary
9. Optionally writes a `.loading` file for the loading screen
