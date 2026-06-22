# Memory Cards

SplashEdit can write real **memory card saves** — Sony-format save files that survive a power-off and show up in the PlayStation BIOS memory card manager, complete with your own title and icon.

This is different from [`Persist`](../lua/api-reference.md#persist), which only keeps data in RAM until the console is switched off. Use `Persist` for cross-scene state during a session; use memory cards for actual save games.

| | `Persist` | `MemCard` |
|---|-----------|-----------|
| Survives scene load | Yes | Yes |
| Survives power-off | No | Yes |
| Shows in BIOS | No | Yes |
| Stores | Numbers only, 16 keys | Whole Lua tables (nested) |
| Speed | Instant | Blocking disk access |

## Project Setup

Memory card settings are **global to the project** (not per-scene). They are packed into every scene's splashpack so the runtime can build standards-compliant save files.

1. Open the **SplashEdit Control Panel** (**PlayStation 1 -> SplashEdit Control Panel**).
2. Go to the **Memory Card** tab.
3. Tick **Enable Saves**.
4. Fill in the fields below.

| Field | Description |
|-------|-------------|
| Enable Saves | Packs the memory card configuration into the splashpack. Off by default. |
| Region Code | Two letters: `BA` = America, `BE` = Europe, `BI` = Japan. Used to build the save filename. |
| Product Code | Up to 10 characters, e.g. `SLUS-00000`. Combined with the region and your save key to form the Sony filename. |
| Save Title | Shown in the BIOS memory card manager (ASCII, up to 32 characters). Can be overridden per-save from Lua. |
| Icon Frames (16x16) | 1 = static icon, 2-3 = animated. Each frame is a 16x16 texture. Frames should share frame 0's palette. |

!!! tip "Sensible defaults"
    If a splashpack has no memory card configuration (for example an older project, or saves left disabled), the runtime falls back to built-in defaults so `MemCard.Save` still works out of the box — you just get a generic title and a blank icon.

!!! note "Icon palette"
    The BIOS icon is a 16-colour (4bpp) image. All animation frames share a single palette taken from the first frame, so design your frames against the same set of colours.

## Saving and Loading from Lua

The [`MemCard` API](../lua/api-reference.md#memcard) is intentionally explicit: `port` is `0` (slot 1) or `1` (slot 2), and **every call returns a result plus an error string** so you always know what happened.

```lua
-- Save the current game state to slot 1
function saveGame()
    local state = {
        level = Scene.GetIndex(),
        hp = FixedPoint.new(100),
        name = "HERO",
        flags = { metWizard = true, bridgeRepaired = false },
    }

    local ok, err = MemCard.Save(0, "save1", state)
    if ok then
        setStatus("Game saved!")
    else
        setStatus("Save failed: " .. err)
    end
end
```

```lua
-- Load it back
function loadGame()
    local data, err = MemCard.Load(0, "save1")
    if not data then
        setStatus("No save found: " .. err)
        return
    end
    Scene.Load(data.level)
    -- data.hp is a FixedPoint, data.name is a string, data.flags is a table...
end
```

### What you can store

A save is one Lua value (usually a table). Inside it you may use:

- `nil`, booleans
- integer numbers
- strings
- `FixedPoint` values
- nested tables (up to **16 levels** deep)

Functions, threads and other userdata are rejected with an error. Cyclic tables are rejected by the depth limit. The blob is length-prefixed and checksummed, so a corrupt or foreign file fails to load rather than being silently misread.

### A safe save flow

```lua
function trySave(port, key, data)
    local present, err = MemCard.IsPresent(port)
    if not present then
        return false, "No memory card in slot " .. (port + 1)
    end

    local blocks = MemCard.FreeBlocks(port) or 0
    if blocks < 1 then
        return false, "Memory card is full"
    end

    return MemCard.Save(port, key, data)
end
```

### Listing and deleting saves

```lua
-- Show what's on the card
local names = MemCard.List(0)
if names then
    for i = 1, #names do
        Debug.Log("Save: " .. names[i])
    end
end

-- Delete a save
MemCard.Delete(0, "save1")
```

!!! danger "Format erases everything"
    `MemCard.Format(port)` writes a fresh, empty filesystem to the card, **wiping all saves on it** — including saves from other games. Only call it behind an explicit player confirmation.

## Important Notes

!!! warning "Saving is blocking"
    `Save`, `Load`, `Format`, `List` and `FreeBlocks` block while the card is accessed (tens of milliseconds, sometimes longer on a busy card). Trigger them at deliberate save points — a save room, a menu, a checkpoint — never inside `onUpdate`.

!!! note "Slots and cards"
    Port `0` is controller/memory-card slot 1, port `1` is slot 2. A standard card holds 15 blocks of 8 KiB. Most saves fit comfortably in a single block.

See the [`MemCard` API reference](../lua/api-reference.md#memcard) for the full per-function signatures and return values.
