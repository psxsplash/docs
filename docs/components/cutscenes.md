# Cutscenes

Cutscenes are pre-authored sequences of keyframed animations with camera control and audio events. Only **one cutscene can play at a time**. For multi-instance playback without camera control, see [Animations](animations.md).

## Creating a Cutscene

1. In the Project window, right-click and select **Create -> PSXSplash -> Cutscene Clip**
2. This creates a `PSXCutsceneClip` ScriptableObject
3. Add it to the [Scene Exporter's](scene-exporter.md) **Cutscenes** array

## PSXCutsceneClip Settings

| Field | Description |
|-------|-------------|
| Cutscene Name | Max 24 chars, unique per scene. Used in Lua: `Cutscene.Play("name")` |
| Duration Frames | Total length in frames at 30fps (150 = 5 seconds) |
| Tracks | Array of animation tracks (see below) |
| Audio Events | Array of timed audio triggers (see below) |

## Track Types

Each track targets a specific property:

| Track Type | Target | Value Meaning |
|-----------|--------|---------------|
| Camera Position | Camera | World-space XYZ |
| Camera Rotation | Camera | Rotation in pi-units |
| Object Position | Named GameObject | World-space XYZ |
| Object Rotation | Named GameObject | Rotation in pi-units |
| Object Active | Named GameObject | Show/hide (step only) |
| UI Canvas Visible | Named canvas | Show/hide (step only) |
| UI Element Visible | Named element | Show/hide (step only) |
| UI Progress | Named element | Progress bar value |
| UI Position | Named element | Screen-space XY |
| UI Color | Named element | RGB color |

For **Object** tracks, the Object Name field must match the GameObject's name in the scene.
For **UI** tracks, set the Canvas Name and Element Name to match your [UI setup](ui.md).

## Keyframes

Each track has an array of keyframes:

| Field | Description |
|-------|-------------|
| Frame | Frame number (0 = cutscene start) |
| Value | Vector3 value (interpretation depends on track type) |
| Interp | Interpolation mode |

### Interpolation Modes

| Mode | Description |
|------|-------------|
| Linear | Constant speed between keyframes |
| Step | Jump instantly to the new value at the keyframe |
| EaseIn | Start slow, accelerate (cubic) |
| EaseOut | Start fast, decelerate (cubic) |
| EaseInOut | Slow at both ends, fast in the middle |

## Audio Events

Timed audio triggers within the cutscene:

| Field | Description |
|-------|-------------|
| Frame | When to play the sound |
| Clip Name | Must match a [PSXAudioClip's](audio.md) ClipName in the scene |
| Volume | 0-128 |
| Pan | 0-127 (0=left, 64=center, 127=right) |

## Lua Playback

```lua
-- Simple play
Cutscene.Play("intro")

-- Play with options
Cutscene.Play("camera_flyover", {
    loop = false,
    onComplete = function()
        Controls.SetEnabled(true)
        setStatus("Cutscene done!")
    end
})

-- Play looping (e.g., ambient animation)
Cutscene.Play("ambient_spin", {loop = true})

-- Stop the current cutscene
Cutscene.Stop()

-- Check if any cutscene is playing
if Cutscene.IsPlaying() then
    Debug.Log("Cutscene running")
end
```

!!! note "Callback timing"
    The `onComplete` callback fires once when the cutscene finishes. For looping cutscenes, it fires when `Cutscene.Stop()` is called, not on each loop.

## Editor Preview

The cutscene inspector has **play/preview controls**. Click the play button to scrub through the cutscene in the Scene view. Camera tracks move the Scene view camera during preview. The original camera position is restored when preview ends.

## Limits

| Resource | Limit |
|----------|-------|
| Cutscenes per scene | 16 |
| Tracks per cutscene | 8 |
| Keyframes per track | 64 |
| Audio events per cutscene | 64 |

## Cutscenes vs Animations

| Feature | Cutscene | Animation |
|---------|----------|-----------|
| Camera tracks | Yes | No |
| Audio events | Yes | No |
| Simultaneous playback | One at a time | Up to 8 |
| Same clip stacking | No | Yes |
| API | `Cutscene.*` | `Animation.*` |

Use cutscenes for camera-driven sequences. Use [animations](animations.md) for everything else.
