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
| Duration (s) | Total length in seconds. Stored internally as frames at 30fps (e.g., 5s = 150 frames). |
| Tracks | Array of animation tracks (see below) |
| Audio Events | Array of timed audio triggers (see below) |
| Skin Anim Events | Timed triggers for [skinned mesh animations](skinned-meshes.md) (see below) |

## Track Types

Each track targets a specific property:

| Track Type | Target | Value Meaning |
|-----------|--------|---------------|
| Camera Position | Camera | World-space XYZ |
| Camera Rotation | Camera | Rotation in pi-units |
| Camera H | Camera | Projection distance (FOV control), X component only |
| Object Position | Named GameObject | World-space XYZ |
| Object Rotation | Named GameObject | Rotation in pi-units |
| Object Active | Named GameObject | Show/hide (step only) |
| UI Canvas Visible | Named canvas | Show/hide (step only) |
| UI Element Visible | Named element | Show/hide (step only) |
| UI Progress | Named element | Progress bar value |
| UI Position | Named element | Screen-space XY |
| UI Color | Named element | RGB color |
| Rumble Small | Controller (global) | On/off (step only) |
| Rumble Large | Controller (global) | Motor speed 0–255 (interpolated) |

For **Object** tracks, the Object Name field must match the GameObject's name in the scene.
For **UI** tracks, set the Canvas Name and Element Name to match your [UI setup](ui.md).

## Keyframes

Each track has an array of keyframes:

| Field | Description |
|-------|-------------|
| Frame | Frame number (0 = cutscene start) |
| Value | Vector3 value (interpretation depends on track type; for Camera H only the X component is used) |
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

## Skin Anim Events

Timed triggers for [skinned mesh animations](skinned-meshes.md) within the cutscene. Works like audio events — specify a time, target, and clip:

| Field | Description |
|-------|-------------|
| Time (s) | When to trigger the skinned animation |
| Target Object | Name of the target object (must have a `PSXSkinnedObjectExporter`) |
| Clip Name | Name of the animation clip on the target |
| Loop | Whether the triggered animation should loop |

The editor validates that the target object exists in the scene and has the specified clip. Events are processed in frame order during playback.

!!! tip
    Combine skin anim events with camera tracks for cinematic sequences — e.g., cut to a character, trigger their animation, and move the camera simultaneously.

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

## Time-Based Playback

Cutscenes use time-based advancement internally (0.12 fixed-point delta time). Playback speed is consistent regardless of the actual framerate. The editor displays all timing in seconds.

## Editor Preview

Cutscenes can be edited in the **PSX Timeline** editor window (**PlayStation 1 > Timeline Editor** from the menu, or double-click a cutscene clip asset). The timeline provides:

- A visual track layout with camera, object, and UI tracks
- Audio event and skin anim event editing on the timeline
- Drag-and-drop keyframe editing with interpolation mode selection
- Play/scrub controls to preview the cutscene in the Scene view
- Camera tracks move the Scene view camera during preview (original position restored when preview ends)
- Targeted skinned meshes show the correct pose at the current time

You can also preview directly from the cutscene inspector by clicking the play button.

## Limits

| Resource | Limit |
|----------|-------|
| Cutscenes per scene | 16 |
| Tracks per cutscene | 8 |
| Keyframes per track | 64 |
| Audio events per cutscene | 64 |
| Skin anim events per cutscene | 16 |

## Controller Rumble

Cutscenes can drive DualShock controller vibration via two dedicated track types:

| Track | Motor | Values | Behavior |
|-------|-------|--------|----------|
| **Rumble Small** | Right motor (high-frequency) | 0 = off, non-zero = on | Step only — snaps on/off at each keyframe |
| **Rumble Large** | Left motor (low-frequency) | 0–255 motor speed | Interpolated — smoothly ramps between keyframes |

Rumble tracks are global (no target object needed). Add keyframes to control when and how strongly the controller vibrates during the cutscene. Motors are automatically stopped when the cutscene ends.

!!! tip "Cinematic rumble"
    Pair rumble with camera shakes and audio events for impactful cutscene moments — explosions, boss introductions, dramatic falls.

!!! note
    Rumble requires a DualShock controller (or emulator rumble support). Digital controllers ignore vibration data.

## Cutscenes vs Animations

| Feature | Cutscene | Animation |
|---------|----------|-----------|
| Camera tracks (position, rotation, FOV) | Yes | No |
| Audio events | Yes | No |
| Skin anim events | Yes | Yes |
| Controller rumble tracks | Yes | Yes |
| Simultaneous playback | One at a time | Up to 8 |
| Same clip stacking | No | Yes |
| API | `Cutscene.*` | `Animation.*` |

Use cutscenes for camera-driven sequences. Use [animations](animations.md) for everything else.
