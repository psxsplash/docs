# Skinned Meshes

Skinned meshes allow bone-based character animation on PS1. Attach a `PSXSkinnedObjectExporter` to any GameObject with a `SkinnedMeshRenderer`, assign animation clips, and play them at runtime via [Lua](#lua-playback) or as timed events inside [cutscenes](cutscenes.md) and [animations](animations.md).

## How It Works

At export time, SplashEdit:

1. Creates a temporary bind-pose mesh proxy for the skinned object (this is what gets exported as normal geometry)
2. **Bakes** every animation clip by sampling bone transforms at the configured FPS
3. Stores per-frame bone matrices (3×3 rotation + translation) in the splashpack binary
4. Records per-triangle bone indices (hard skinning — each vertex uses its highest-weight bone)

At runtime on PS1, the engine:

1. Composes camera × object × bone matrices per bone
2. Transforms each vertex individually using its assigned bone's matrix via the GTE (`rtps`)
3. Interpolates between baked frames with sub-frame precision for smooth playback at any framerate

!!! note "Hard skinning"
    The PS1 doesn't have the power for multi-bone blending per vertex. Each vertex is assigned to a **single bone** (the one with the highest weight). Design your meshes with this in mind — weight painting should use clean, hard transitions between bones.

## Adding a Skinned Object

1. Import your rigged model (FBX recommended)
2. Place it in the scene — it should have a `SkinnedMeshRenderer`
3. Add the `PSXSkinnedObjectExporter` component to the same GameObject
4. Assign animation clips to the **Animation Clips** array. Remember if you can't preview them in Unity they won't work on the PSX.

## PSXSkinnedObjectExporter Settings

| Field | Description | Default |
|-------|-------------|---------|
| Animation Clips | Array of `AnimationClip` assets to bake. Each becomes a named clip playable via `SkinnedAnim.Play()` | Empty |
| Target FPS | Bone matrix sampling rate (1–30). Lower = less memory, higher = smoother. 15 is usually sufficient. | 15 |
| Is Active | Whether the object starts active in the scene | On |
| Lua File | Optional [Lua script](../lua/index.md) for this object's behavior | None |
| Bit Depth | Texture color depth: 4-bit, 8-bit, or 16-bit | 8-bit |
| Color Mode | Vertex color mode: Baked Lighting, Flat Color, or Mesh Vertex Colors | Baked Lighting |
| Flat Vertex Color | (Flat Color mode only) Solid color applied to all vertices | 128, 128, 128 |
| Smooth Normals | Average normals across shared vertices for smooth lighting. Disable for flat/faceted shading. | On |

## Lua Playback

Use the `SkinnedAnim` API to control skinned animations from Lua scripts. See the [Lua API Reference](../lua/api-reference.md#skinnedanim) for the full API.

```lua
-- Play a clip by object name and clip name
SkinnedAnim.Play("MyCharacter", "walk", { loop = true })

-- Play one-shot with callback
SkinnedAnim.Play("MyCharacter", "attack", {
    onComplete = function()
        SkinnedAnim.Play("MyCharacter", "idle", { loop = true })
    end
})

-- Stop playback
SkinnedAnim.Stop("MyCharacter")

-- Check if playing
if SkinnedAnim.IsPlaying("MyCharacter") then
    Debug.Log("Animating")
end

-- Get current clip name
local clip = SkinnedAnim.GetClip("MyCharacter")
```

## Skin Anim Events

Skinned animations can also be triggered at specific times during [cutscenes](cutscenes.md) and [animations](animations.md) using **Skin Anim Events**. This works the same way as audio events - you specify a trigger time, target object, clip name, and whether it should loop.

See [Cutscenes — Skin Anim Events](cutscenes.md#skin-anim-events) and [Animations — Skin Anim Events](animations.md#skin-anim-events).

## Export Details

During scene export, each `PSXSkinnedObjectExporter` goes through a special pipeline:

1. A **proxy** `PSXObjectExporter` is created from the bind-pose mesh 
2. The original `SkinnedMeshRenderer` object's polygons are "stolen" by the skinned mesh system - the normal renderer skips it
3. Per-bone matrices are baked for every frame of every clip at the configured FPS
4. Bone indices (3 per triangle - one per vertex) are stored for the renderer to look up per-vertex bone transforms

The proxy is destroyed after export completes.

## Memory Cost

Skinned mesh data uses significantly more memory than static objects due to the per-frame bone matrices:

**Per frame per bone**: 24 bytes (9×int16 rotation + 3×int16 translation)

**Example**: A character with 20 bones and a 2-second walk cycle at 15fps:

- 30 frames × 20 bones × 24 bytes = **14,400 bytes** per clip

Total memory scales with: `clips × frames × bones × 24 bytes + triangles × 3 bytes (bone indices)`

!!! tip "Reduce memory"
    - Lower **Target FPS** (15 is visually smooth enough for most animations thanks to sub-frame interpolation)
    - Use fewer bones (10–20 is typical for PS1 characters)
    - Keep clips short (reuse idle/walk loops)
    - The runtime interpolates between frames, so you can get away with surprisingly low sample rates

## Limits

| Resource | Limit |
|----------|-------|
| Skinned meshes per scene | 16 |
| Bones per mesh | 64 |
| Clips per mesh | 16 |
| Clip name length | 24 characters |
| Frame count per clip | No hard cap (memory dependent) |

## Tips

!!! tip "Weight painting"
    Since the PS1 uses hard skinning (one bone per vertex), make sure your weight painting has clean, clear bone assignments. Avoid soft blends across many bones — the engine picks only the strongest weight.

!!! tip "Animation clip names"
    Clip names are used in Lua to reference animations. They come from the Unity AnimationClip asset name. Keep them short and descriptive: `idle`, `walk`, `attack`, `jump`.

