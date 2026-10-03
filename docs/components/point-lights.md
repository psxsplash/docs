# Point Lights

Point lights light your meshes with a coloured, fading glow. They can be baked into the mesh at export, or run live on the PS1 so they can move, change colour and switch on and off from Lua.

!!! info "Next release"
    Runtime Point Lights arrive in the next release. They are not in 2.4.0.

## Setup

Use Unity's normal **Point Light**. There is no extra component. The light's **Light Mode** decides how it is exported:

| Light Mode | Result |
|------------|--------|
| Baked | Baked into vertex colors at export, as before. Free at runtime. See [Objects & Meshes](objects.md#mesh-conversion). |
| Realtime or Mixed | A runtime light on the PS1. It can move, change color and switch on or off from Lua. |

| Unity field | Effect |
|-------------|--------|
| Range | The light's radius. Falls off linearly to zero at the range. |
| Color | Maps directly. |
| Intensity | Maps directly. Capped just under 16. |

!!! warning "Existing scenes"
    A scene that used Point Lights in **Realtime** mode for baking will now light them at runtime instead. Set them to **Baked** to keep the old look.

## Which Meshes Are Lit

On each [PSXObjectExporter](objects.md), **Dynamic Lighting** controls whether the mesh is lit by runtime lights:

| Setting | Behavior |
|---------|----------|
| Auto (default) | Lit if a runtime light's range reaches the mesh at export. |
| On | Always lit. Use for meshes that move into lights. |
| On (smooth) | Lit per vertex. Smoother on big triangles, at about 9x the cost. |
| Off | Never lit at runtime. Runtime lights are baked into this mesh instead. |

## Limits

- 4 runtime lights per mesh. Extra lights are ignored, with a warning naming them.
- 16 runtime lights per scene.
- Skinned meshes are not lit at runtime.

## Cost

Default lighting is per triangle (flat), so large triangles look faceted. Every lit triangle costs extra CPU time.

!!! tip
    Keep the range of runtime lights small and lit meshes modest. A light that never moves should be **Baked**.

## Editor Help

- The PSXObjectExporter inspector lists the runtime lights reaching the mesh, and whether it ends up runtime-lit or baked.
- Turn on **Preview Point Lights** on the [Scene Exporter](scene-exporter.md) to draw lines from the selected mesh to its lights. Yellow lines are used, red lines are dropped.
- A red box marks meshes over the 4-light cap.

## Controlling Lights from Cutscenes and Animations

[Cutscenes](cutscenes.md#light-tracks) and [animations](animations.md) can keyframe a runtime light's position, color, intensity, range and on/off state. Baked lights cannot be targeted. See [Light Tracks](cutscenes.md#light-tracks).

## Controlling Lights from Lua

```lua
local lamp = Light.Find("Lamp")
Light.SetColor(lamp, 255, 80, 40)
Light.SetEnabled(lamp, false)
```

See the [Light API](../lua/api-reference.md#light).
