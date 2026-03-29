# Lua Scripting

Lua is the scripting language for all game logic in SplashEdit. Scripts control object behavior, UI, audio, scene transitions, and more.

## Important: This is NOT Standard Lua

The Lua VM running on PS1 uses **fixed-point numbers** instead of floating-point. The PS1 has no floating-point hardware. This changes how you write math:

- **There are no decimal numbers.** `0.5` does not exist.
- **Integer division gives integers.** `1/2` equals `0`, not `0.5`.
- **Use `FixedPoint.new()` for ALL fractional values.** This is the only way to get non-integer numbers.

Read [Fixed-Point Math](fixed-point.md) before writing any scripts. This is the single most important thing to understand.

## Script Types

There are three types of Lua scripts:

### Scene Scripts

Attached to the [Scene Exporter](../components/scene-exporter.md). Run during scene initialization. This is where you define shared functions, set up UI, and initialize game state.

### Object Scripts

Attached to individual [PSXObjectExporter](../components/objects.md) components. Define per-object behavior through [event callbacks](events.md) like `onInteract`, `onCollideWithPlayer`, and `onButtonPress`.

### Trigger Scripts

Attached to [PSXTriggerBox](../components/trigger-boxes.md) components. Handle `onTriggerEnter` and `onTriggerExit` events.

## Creating Lua Scripts

1. Create a `.lua` file in your Unity project's Assets folder
2. Unity imports it automatically as a `LuaFile` ScriptableObject
3. Assign the LuaFile to a Scene Exporter, Object Exporter, or Trigger Box

## Script Environments

Each script runs in **its own environment**. Functions defined in one script are NOT automatically available to other scripts. To share functions (for example, from a scene script to object scripts), publish them to `_G`:

```lua
-- In scene.lua
function addScore(amount)
    -- ...
end

-- Publish so object scripts can call it
_G.addScore = addScore
```

Then any object script can call `addScore(100)` directly, because `_G` is the fallback for all environments.

## What's in This Section

1. [**Fixed-Point Math**](fixed-point.md) - How numbers work on PS1 (read this first!)
2. [**Event Callbacks**](events.md) - All available callbacks and when they fire
3. [**API Reference**](api-reference.md) - Complete reference for every Lua function
4. [**Patterns & Best Practices**](patterns.md) - Working code patterns for common game mechanics
