# Tutorial: Animated Doors

A door that opens and closes on a button press, using an animation built in the timeline.

You will use the **Timeline**, an **Interactable**, `Animation.Play` with `onComplete`, and `Animation.IsPlaying`.

## 1. The door

Add a **Mesh** object for the door and name it exactly `Door`. The name is how the animation finds it.

## 2. The animations

1. Open **View > Timeline**, choose **New animation**, and name it `door_open`.
2. Set **Length** to 30 (one second).
3. **Add track**: **Object > Position**, target `Door`.
4. Scrub to frame 0 and double-click the lane to key the door's closed position. Scrub to frame 30, move the door where it should end up (for example 3 m up), and key it there.
5. Make a second animation, `door_close`, with the same two keys reversed.

## 3. Interaction

On the Door, **Add component > Interactable**. The defaults work: Cross button, radius 2 m, repeatable. Turn on **Show prompt** and pick a canvas if you have one with a "Press X" message.

## 4. Sounds

Add two **Audio clip** objects named `door_open` and `door_close`, each with a WAV.

## 5. The script

On the Door, **New Lua** in the Script component:

```lua
local isOpen = false

function onInteract(self)
    -- Ignore presses while an animation runs
    if Animation.IsPlaying("door_open") or Animation.IsPlaying("door_close") then
        return
    end

    Interact.SetEnabled(self, false)

    if isOpen then
        isOpen = false
        Audio.Play("door_close", 100, 64)
        Animation.Play("door_close", {
            onComplete = function() Interact.SetEnabled(self, true) end
        })
    else
        isOpen = true
        Audio.Play("door_open", 100, 64)
        Animation.Play("door_open", {
            onComplete = function() Interact.SetEnabled(self, true) end
        })
    end
end
```

## 6. Play

Walk up and press Cross to open; press again to close. You cannot interrupt the animation, the prompt hides while it plays, and `onComplete` turns interaction back on.

## How it works

1. `Animation.IsPlaying` stops overlapping animations.
2. `Interact.SetEnabled(self, false)` hides the prompt and blocks presses.
3. `Animation.Play` starts the animation and calls `onComplete` when it ends.
4. `isOpen` remembers which way to go next.

!!! note "No collision"
    This door has no collider, so the player can walk through it open or shut. A Static collider feeds the navigation bake at export, and a door that slides away later does not change that bake. To keep the player out, put a Trigger box or an invisible wall where the door is, and treat the door as scenery you animate.
