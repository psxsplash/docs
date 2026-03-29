# Interactables

Interactables are objects the player can interact with by pressing a button when nearby.

## Setup

1. The object must have a `PSXObjectExporter` component
2. Add a `PSXInteractable` component to the **same GameObject**

## PSXInteractable Settings

| Field | Default | Description |
|-------|---------|-------------|
| Interaction Radius | 2.0 | How close the player must be to interact |
| Interact Button | Cross | PS1 button (dropdown selector in the inspector) |
| Is Repeatable | true | Can the player interact more than once? |
| Cooldown Frames | 30 | Minimum frames between interactions |
| Show Prompt | false | Show a UI canvas when in range |
| Prompt Canvas Name | "" | Which canvas to show as a prompt (max 15 chars) |
| Require Line of Sight | false | Must the player face the object? |

The button selector is a dropdown in the Unity inspector - you don't need to know button codes.

## Lua Integration

When the player presses the interact button within range, the object's Lua script receives an `onInteract(self)` callback:

```lua
function onInteract(self)
    Debug.Log("Player interacted!")
    -- Do something
end
```

### Enabling/Disabling Interactions

You can toggle interactions from Lua. This also hides the prompt canvas if one is configured:

```lua
-- Disable interaction (e.g., during an animation)
Interact.SetEnabled(self, false)

-- Re-enable
Interact.SetEnabled(self, true)

-- Check state
if Interact.IsEnabled(self) then
    -- ...
end
```

### Common Pattern: Disable During Animation

```lua
function onInteract(self)
    if Animation.IsPlaying("door_open") then return end

    Interact.SetEnabled(self, false)
    Animation.Play("door_open", {
        onComplete = function()
            Interact.SetEnabled(self, true)
        end
    })
end
```

## Prompt Canvases

If **Show Prompt** is enabled and a **Prompt Canvas Name** is set, SplashEdit automatically shows/hides that [UI canvas](ui.md) when the player enters/exits the interaction radius.

This is useful for "Press X to interact" prompts.

## Gizmo

The `PSXInteractable` draws a **yellow semi-transparent sphere** in the Scene view showing the interaction radius.
