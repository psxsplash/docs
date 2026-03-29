# Tutorial: Animated Doors

Create a door that opens and closes with animation, sound, and interaction gating.

## What You'll Learn

- `Animation.Play` with `onComplete` callbacks
- `Animation.IsPlaying` to prevent overlapping animations
- `Interact.SetEnabled` to disable interaction during animation
- State tracking with boolean flags

## Step 1: Create Animation Clips

1. **Create "door_open"**: Right-click in Project -> Create -> PSXSplash -> Animation Clip
    - Name: `"door_open"`
    - Duration: 30 frames (1 second at 30fps)
    - Add one track: **Object Position**, Object Name: `"Door"`
    - Keyframe 0: Current door position (e.g., 0, 0, 0), Linear
    - Keyframe 30: Open position (e.g., 0, 3, 0 to slide up, or 2, 0, 0 to slide sideways)

2. **Create "door_close"**: Same as above but reversed keyframe positions
    - Keyframe 0: Open position
    - Keyframe 30: Closed position

3. Add both clips to the Scene Exporter's **Animations** array

## Step 2: Door Script

Create `door.lua`:

```lua
local isOpen = false

function onCreate(self)
    isOpen = false
    Debug.Log("Door created at "
        .. self.position.x .. "," .. self.position.y .. "," .. self.position.z)
end

function onInteract(self)
    -- Prevent interaction while animating
    if Animation.IsPlaying("door_open") or Animation.IsPlaying("door_close") then
        return
    end

    -- Disable interact prompt during animation
    Interact.SetEnabled(self, false)

    if isOpen then
        isOpen = false
        Audio.Play("door_close", 100, 64)
        setStatus("Closing door...")

        Animation.Play("door_close", {
            onComplete = function()
                Interact.SetEnabled(self, true)
                setStatus("Door closed.")
            end
        })
    else
        isOpen = true
        Audio.Play("door_open", 100, 64)
        setStatus("Opening door...")

        Animation.Play("door_open", {
            onComplete = function()
                Interact.SetEnabled(self, true)
                setStatus("Door opened!")
            end
        })
    end
end
```

## Step 3: Scene Setup

1. Create a door mesh GameObject named exactly **"Door"** (must match the animation track's Object Name)
2. Add `PSXObjectExporter`, assign `door.lua`
3. Add `PSXInteractable` (default settings work: Cross button, radius 2.0)

## Step 4: Add Audio

Add `PSXAudioClip` components for `"door_open"` and `"door_close"` with appropriate sounds.

## Step 5: Build and Test

Walk up to the door, press Cross to open. Press again to close. Notice that:

- You can't spam-interact during animation (gating works)
- The prompt disappears during animation
- Sound plays at the start of each animation
- The `onComplete` callback re-enables interaction

## How It Works

1. `Animation.IsPlaying` check prevents overlapping animations
2. `Interact.SetEnabled(self, false)` hides the prompt and blocks further interaction
3. `Animation.Play` starts the animation and registers a completion callback
4. When the animation finishes, the callback re-enables interaction
5. The `isOpen` flag tracks the door state for toggling

