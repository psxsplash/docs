# Lighting

The PS1 has no per-pixel lighting. psxsplash lights a mesh in one of two ways: colours **baked** into its vertices at export, or a few **runtime** point lights added per triangle on the console.

## The Light component

Add one from **Add Object > Point light**, or **Add component > Light** on any object.

| Field | Meaning |
|-------|---------|
| **Type** | **Point** shines in every direction, **Spot** in a cone, **Directional** from far away (like the sun). |
| **Colour** | The light's colour. |
| **Intensity** | How bright it is. 1 is normal. |
| **Range** | How far it reaches before fading out. Point and spot only. |
| **Spot angle**, **Inner angle** | Spot only. Inside the inner cone the light is at full strength and fades out to the spot angle. 0 fades from 80% of the spot angle. |

The switch on the component header turns it off. The viewport's **PS1** view shows the baked result on the walls and floor, so you see a light's effect as you move it.

## Baked lighting

A mesh with **Lighting: Baked vertex** (the default) gets the scene's lights baked into its vertex colours when the scene exports. It costs nothing at runtime, and it is what the courtyard example uses. Triangles are lit per vertex, so large triangles shade smoothly only if the mesh has vertices to carry it: split big floors and walls if a light looks blotchy.

**Smooth normals** on the mesh changes whether the bake treats surfaces as flat or curved. See [Materials & Bit Depth](materials.md#mesh-colours-and-lighting).

## Runtime lights

A **point** light can instead run on the console, so scripts can move it, recolour it and switch it on and off with the [`Light` API](../lua/api-reference.md#light), and [timeline tracks](timeline.md#tracks) can animate it. In the scene file this is `"runtime": true` on the light component (see the scene format notes in the editor's repository); the inspector does not show a switch for it yet.

- At most **16** runtime lights per scene.
- At most **4** reach one mesh. Extra lights are left out.
- Skinned meshes are not lit at runtime.
- A mesh lit at runtime leaves runtime lights out of its baked colours; every other mesh bakes them in.

### Dynamic lighting

Which meshes runtime lights affect is set per mesh and starts from the model's import settings, **Lighting for new objects**:

| Setting | Behaviour |
|---------|-----------|
| **Auto** | Lit if a runtime light's range reaches the mesh at export. |
| **On** | Always lit. Use it for meshes that move into a light's range. |
| **Smooth** | Lit per vertex. Smoother on big triangles, at about nine times the cost. |
| **Off** | Never lit at runtime. |

Every lit triangle costs CPU time. Keep ranges small, and leave lights that never move baked.
