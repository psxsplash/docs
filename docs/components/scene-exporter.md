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
| Fog Color | RGB color for distance fog. Also used as the background/clear color. | Black |
| Fog Enabled | Toggle fog on/off | Off |
| Fog Density | How quickly fog appears with distance (higher = closer fog). Uses a Silent Hill-style two-pass approach with per-vertex fog blending and additive overlay. | 0 |
| Cutscenes | Array of [PSXCutsceneClip](cutscenes.md) assets used in this scene | Empty |
| Animations | Array of [PSXAnimationClip](animations.md) assets used in this scene | Empty |
| Loading Screen Prefab | Optional prefab shown while the scene [loads](loading-screens.md) | None |

## Scene Types

### Exterior

Uses a **Bounding Volume Hierarchy (BVH)** for frustum culling. The BVH groups triangles spatially and culls entire branches that fall outside the camera's view frustum. Objects moved via `Entity.SetPosition` are automatically marked as dynamically moved and bypass the static BVH, using their shifted AABB for frustum culling instead.

Best for: open areas, outdoor environments.

!!! warning
    Interior scene type with rooms and portals is still recommended for best performance. Exterior BVH mode works well for most scenes but interior mode gives tighter culling.

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
2. Discovers `PSXSkinnedObjectExporter` components and creates temporary bind-pose proxy objects
3. Quantizes and deduplicates textures, packs into VRAM
4. Converts meshes to PS1 format with vertex colors (baked lighting, flat color, or mesh vertex colors depending on each object's settings)
5. Bakes skinned mesh bone matrices for every animation clip at the configured FPS
6. Builds BVH (exterior) or room/portal structure (interior)
7. Generates navigation regions from walkable geometry
8. Exports UI canvases, custom fonts, and font pixel data
9. Exports cutscenes, animations (with skin anim events), audio, interactables, and trigger boxes
10. Writes skinned mesh data (bone indices, baked frames) to the splashpack binary
11. Writes the complete `.splashpack` binary
12. Optionally writes a `.loading` file for the loading screen
13. Destroys temporary skinned mesh proxy objects
