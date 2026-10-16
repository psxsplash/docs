# Skinned Animation

A skinned mesh is a model with a skeleton. The PS1 skins it **rigidly**: each vertex follows the one joint that influences it most, and the engine draws at most 64 joints per model.

## Getting one in

Import a glTF or FBX with a skin (see [Importing](importing.md)). The editor keeps the skeleton in the `.mesh` and writes one `models/<name>/<clip>.anim` file per animation in the model file. Each clip's `name` is what Lua plays it by, 1 to 24 characters and unique per object. FBX animations are baked at 30 Hz.

Place the model, then **Add component > Skinned animation** (it needs a Mesh with a skeleton first).

## The component

| Field | Meaning |
|-------|---------|
| **Clips** | The `.anim` files this object can play, at most 16. Click to pick them; the pick replaces the list. |
| **Bake rate** | Frames per second the clips are sampled at, 1 to 30. Default 15. The engine blends between frames, so a low rate still looks smooth. |
| **Baked** | What the export makes of the clips at this bake rate: the bones and triangles, then each clip's frame count and size in KB, and the skin data total. |

A **Long clip** card appears when a clip bakes to more than 1000 frames; lower the bake rate or trim the clip. A looping clip stores its frames over `[0, length)` and wraps from the last frame to the first, so the end pose is not stored twice.

## Playing clips

```lua
SkinnedAnim.Play("Hero", "walk", { loop = true })
SkinnedAnim.Play("Hero", "attack", {
    onComplete = function()
        SkinnedAnim.Play("Hero", "idle", { loop = true })
    end
})
```

The first argument is the **object name**, the second the clip name. A [cutscene or animation](timeline.md#audio-and-skin-events-cutscenes) can start a clip at a frame with a **Skin clip event**. `SkinnedAnim.GetBone` returns where a bone is, to attach things to it. See the [`SkinnedAnim` API](../lua/api-reference.md#skinnedanim).

## Limits

- Skinned meshes are not lit by runtime lights.
- Bones cost RAM per frame of each clip. Fewer bones, shorter clips and a lower bake rate all shrink the scene. See [Memory & Performance](../reference/memory.md).
