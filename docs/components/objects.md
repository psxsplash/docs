# Objects & Meshes

Every renderable object in your scene needs a `PSXObjectExporter` component. This tells SplashEdit to include the object in the export.

## Adding an Object

Three ways:

- Right-click a **MeshFilter** or **MeshRenderer** component and select **Add PSX Object Exporter**
- Use **GameObject -> PlayStation 1 -> Exportable Object**
- Manually add the `PSXObjectExporter` component

The object must have both a **MeshFilter** and a **MeshRenderer** with at least one material.

## Settings

| Field | Description | Default |
|-------|-------------|---------|
| Is Active | Include this object in the export. Uncheck to skip. | On |
| Bit Depth | Texture color depth: 4-bit (16 colors), 8-bit (256 colors), or 16-bit (32768 colors) | 4-bit |
| Lua File | Optional [Lua script](../lua/index.md) for this object's behavior | None |
| Vertex Colors | How vertex colors are computed: Baked Lighting, Flat Color, or Mesh Vertex Colors | Baked Lighting |
| Flat Vertex Color | (Flat Color mode only) Solid color applied to all vertices | 128, 128, 128 |
| Smooth Normals | Average normals across shared vertices for smooth lighting. Disable for flat/faceted shading. | On |
| Is Platform | All boundary edges of nav regions from this mesh allow walkoff. The player can walk off any edge and fall. See [Navigation](navigation.md#platforms). | Off |
| UV Offset Material | Which material/submesh index UV-offset animation applies to. See [UV Offset Animation](#uv-offset-animation). | 0 |
| Collision Type | None, Static, or Dynamic | None |

## Bit Depth

Controls how many colors the object's texture can use. Lower bit depth uses less VRAM.

| Bit Depth | Colors | VRAM Usage | Best For |
|-----------|--------|------------|----------|
| 4-bit | 16 | Very low | Simple textures, solid colors, fonts |
| 8-bit | 256 | Medium | Most game textures |
| 16-bit | 32768 | High | Photographs, gradients |

See [Textures & VRAM](textures.md) for more detail on how textures are packed.

## Collision Types

| Type | Description |
|------|-------------|
| None | No collision. Player walks through. The nav mesh handles walkability. |
| Static | Solid wall. Player cannot walk through. Used for walls, pillars, fixed obstacles. |
| Dynamic | Moveable collider. For objects repositioned via Lua. |

!!! warning "Dynamic collision limitation"
    Rotating objects via Lua does **not** recalculate their AABB (axis-aligned bounding box) collision bounds. The collision volume stays in the original orientation. Position changes via `Entity.SetPosition` will correctly shift the AABB and mark the object for proper frustum culling.

## Mesh Conversion

SplashEdit converts Unity meshes to PS1 format automatically:

- Vertices are converted to **4.12 fixed-point** coordinates (scaled by the Scene Exporter's GTE Scaling)
- Normals are recalculated as smooth normals by default (disable with the **Smooth Normals** toggle for flat/faceted shading)
- **Vertex colors** depend on the Vertex Colors mode:
    - **Baked Lighting** (default): Pre-baked from scene lighting. Material base colors are baked into vertex colors for untextured objects.
    - **Flat Color**: All vertices get the same configurable color (default 128, 128, 128). Useful for unlit objects or stylized looks.
    - **Mesh Vertex Colors**: Uses the vertex colors already present on the mesh (e.g., painted in Blender). Falls back to gray if the mesh has no vertex colors.
- UVs are mapped to the VRAM texture atlas position

## UV Offset Animation

You can scroll an object's texture at runtime by shifting its UV coordinates. This is the classic PS1 trick for flowing water, lava, conveyor belts, waterfalls, force-fields, and simple flipbook effects — all without changing geometry.

There are two ways to drive it:

**1. From a timeline track.** Add an **Object UV Offset** track to a [cutscene](cutscenes.md#track-types) or [animation](animations.md#track-types), targeting the object by name. Each keyframe is an integer `(U, V)` offset in the 0-255 range. The **UV Offset Material** field on the object selects which material/submesh the track scrolls (index `0` is the first material).

**2. From Lua.** Call the runtime [texture functions](../lua/api-reference.md#texture-manipulation):

```lua
-- Scroll the texture horizontally every frame
local u = 0
function onUpdate(self, dt)
    u = (u + 1) % 256
    Entity.SetUVOffset(self, u, 0)   -- absolute, non-destructive
end
```

`Entity.SetUVOffset` applies an absolute offset at render time without touching the stored UVs, so it's safe to animate continuously. `Entity.SetUVs` (additive) and `Entity.SetTPage` (swap texture page) are also available for more advanced effects.

!!! tip "Design textures to tile"
    For seamless scrolling, the texture should tile along the scroll axis. Because offsets wrap at 256, keeping the meaningful texture region within a tile that repeats cleanly avoids visible seams.

## Geometry Guidelines

!!! danger "Don't use Unity's built-in meshes (mostly)"
    Unity's built-in primitives are not designed for PS1:

    - **Plane** - Only 2 huge triangles. Will cause severe texture warping and likely get culled. Never use as a floor.
    - **Sphere, Cylinder, Capsule** - Way too many polygons for PS1.
    - **Cube** - This one is OK for simple testing since it's low-poly.

    **Always create your meshes in a 3D modeling tool** (Blender, Maya, etc.) with PS1 constraints in mind: low polygon count, small triangles, proper UVs.

!!! danger "Texture Requirements"
    All textures **must be power-of-two in both width and height**. Maximum size is **256x256**. SplashEdit does NOT validate this - incorrect textures will produce broken output.

    Valid sizes: 16x16, 32x32, 64x64, 128x128, 256x256, and non-square combinations like 64x128.

## Tips

!!! tip "Triangle counts"
    Keep triangle counts low. The PS1 can handle roughly **1000-2000 visible triangles per frame** at 30fps.

!!! tip "Small triangles"
    The PS1 GPU has a rasterizer limit of 1023 pixels horizontally and 511 vertically between any two vertices of a triangle. Large triangles get subdivided or culled. Keep triangles small by subdividing your meshes in your modeling tool.

!!! tip "Deduplication"
    If multiple objects share the same source texture and bit depth, SplashEdit stores it only once. This happens automatically.
