# Animations

Animations are like [cutscenes](cutscenes.md) but without camera tracks, and **multiple animations can play simultaneously**. Use animations for object movement, UI effects, and anything that doesn't need camera control.

## Creating an Animation

1. Right-click in the Project window -> **Create -> PSXSplash -> Animation Clip**
2. This creates a `PSXAnimationClip` ScriptableObject
3. Add it to the [Scene Exporter's](scene-exporter.md) **Animations** array

## PSXAnimationClip Settings

| Field | Description |
|-------|-------------|
| Animation Name | Max 24 chars, unique per scene. Used in Lua: `Animation.Play("name")` |
| Duration (s) | Total length in seconds. Stored internally as frames at 30fps. |
| Tracks | Array of tracks (Object and UI types only — no Camera, no Audio) |
| Skin Anim Events | Timed triggers for [skinned mesh animations](skinned-meshes.md) (see below) |

!!! note
    If you accidentally add a Camera track to an animation, it will be filtered out with a warning during export.

## Track Types

Same as cutscene tracks, minus camera:

| Track Type | Target |
|-----------|--------|
| Object Position | Named GameObject |
| Object Rotation | Named GameObject |
| Object Active | Named GameObject (step only) |
| UI Canvas Visible | Named canvas |
| UI Element Visible | Named element |
| UI Progress | Named element |
| UI Position | Named element |
| UI Color | Named element |

## Lua Playback

```lua
-- Play one-shot
Animation.Play("door_open")

-- Play with callback
Animation.Play("door_open", {
    onComplete = function()
        Interact.SetEnabled(self, true)
    end
})

-- Play looping
Animation.Play("anim_spinner", {loop = true})

-- Stop by name (stops ALL instances of that animation)
Animation.Stop("anim_spinner")

-- Stop ALL animations
Animation.Stop()

-- Check if any instance is playing
if Animation.IsPlaying("anim_spinner") then
    Debug.Log("Still spinning")
end
```

## Skin Anim Events

Animations can trigger [skinned mesh animations](skinned-meshes.md) at specific times during playback. This works the same way as [cutscene skin anim events](cutscenes.md#skin-anim-events).

| Field | Description |
|-------|-------------|
| Time (s) | When to trigger the skinned animation |
| Target Object | Name of the target object (must have a `PSXSkinnedObjectExporter`) |
| Clip Name | Name of the animation clip on the target |
| Loop | Whether the triggered animation should loop |

```lua
-- Example: door animation triggers a character reaction
Animation.Play("door_open", {
    onComplete = function()
        -- The skin anim event at 0.5s already started the character's
        -- "react" animation, so just play the follow-up:
        SkinnedAnim.Play("NPC", "idle", { loop = true })
    end
})
```

!!! note
    Skin anim events are processed in frame order. Maximum 16 per animation.

## Multi-Instance Playback

Unlike cutscenes, the same animation can play **multiple times simultaneously**. Each `Animation.Play()` call creates a new playback instance. Up to 8 instances can be active at once across all animations.

```lua
-- These create two separate instances, both playing at once
Animation.Play("particle_burst")
Animation.Play("particle_burst")
```

`Animation.Stop("name")` stops **all** instances of that animation.

## Callback Behavior

The `onComplete` callback is stored **per animation name**, not per instance. If the same animation is playing multiple times, the callback fires when any instance of it finishes (and only once per stop, not per-loop).

## Limits

| Resource | Limit |
|----------|-------|
| Animation clips per scene | 16 |
| Tracks per animation | 8 |
| Keyframes per track | 64 |
| Skin anim events per animation | 16 |
| Simultaneous instances | 8 (across all animations) |

## Time-Based Playback

Animations use time-based advancement internally (0.12 fixed-point delta time). This means playback speed is consistent regardless of the actual framerate — if the engine dips below 30fps, animations continue at the correct speed rather than slowing down.

The editor displays all timing in seconds. Internally, 30fps frames are used for storage (e.g., 3 seconds = 90 frames).

## Editor Preview

The animation inspector has play/preview controls for scrubbing through tracks in the Scene view. If the animation has skin anim events, the targeted skinned meshes will also be previewed in the Scene view with the correct pose at the current time.
