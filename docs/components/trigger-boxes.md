# Trigger Boxes

Trigger boxes are invisible volumes that fire Lua events when the player enters or exits them.

## Setup

1. Create an **empty GameObject** where you want the trigger
2. Add a `PSXTriggerBox` component
3. Set the **Size** to define the box dimensions
4. Assign a **Lua File** for the trigger's behavior

## PSXTriggerBox Settings

| Field | Description |
|-------|-------------|
| Size | Box dimensions (width, height, depth) |
| Lua File | Lua script with `onTriggerEnter`/`onTriggerExit` callbacks |

## Creating a Trigger Script

The inspector has a **"Create Lua Script"** button that generates a template `.lua` file with the callback stubs.

## Lua Callbacks

```lua
function onTriggerEnter()
    Debug.Log("Player entered the trigger!")
end

function onTriggerExit()
    Debug.Log("Player left the trigger!")
end
```

!!! important
    Trigger callbacks do **NOT** receive `self` - they have no associated game object. They are standalone scripts attached to the trigger volume.

## Example: One-Time Trigger

```lua
local triggered = false

function onTriggerEnter()
    if triggered then return end
    triggered = true
    setStatus("You found the secret area!")
    Audio.Play("discovery", 100, 64)
end
```

## Example: Damage Zone

```lua
function onTriggerEnter()
    applyDamage(25)  -- Call scene-level function
end
```

## Example: Scene Portal

```lua
function onTriggerEnter()
    Persist.Set("came_from", Scene.GetIndex())
    Scene.Load(1)  -- Load scene 1
end
```

## Example: Cutscene Trigger

```lua
local played = false

function onTriggerEnter()
    if played then return end
    played = true
    Controls.SetEnabled(false)
    Cutscene.Play("camera_flyover", {
        onComplete = function()
            Controls.SetEnabled(true)
        end
    })
end
```

## Tips

!!! tip "One script per trigger"
    Each trigger box has its own Lua script. This is cleaner than routing by index. Give each trigger a descriptive script name.

!!! tip "Sizing"
    The trigger is an axis-aligned box. Position the GameObject where you want the center, and set Size to cover the area. The player triggers it when their position enters the box.
