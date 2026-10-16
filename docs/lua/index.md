# Lua Scripting

Lua runs all game logic in psxsplash. Scripts move objects, show UI, play sounds, run cutscenes and load scenes. You can write the Lua yourself in the [Lua editor](../editor/lua-editor.md), or build the same logic from blocks in the [block editor](../editor/block-editor.md), which writes Lua for you. A scene can mix both.

## This is not standard Lua

The Lua VM on the PS1 uses **fixed-point numbers** instead of floating-point. The PS1 has no floating-point hardware. This changes how you write math:

- **There are no decimal literals.** `0.5` does not exist.
- **Integer division gives integers.** `1/2` equals `0`, not `0.5`.
- **Use `FixedPoint.new()` for fractional values.** It is the only way to get a non-integer number.

Read [Fixed-Point Math](fixed-point.md) before writing scripts. Blocks do this for you: type `0.5` into a block and the generated Lua uses `FixedPoint.new`.

## Where scripts go

There are three places a script attaches, and each has its own events.

| Script | Set in the editor | Gets |
|--------|-------------------|------|
| Scene script | Scene settings, **Script > File** (select nothing to see them) | `onSceneCreationStart`, `onSceneCreationEnd`. Runs once at scene load. A good place for shared functions and game state. |
| Object script | **Script** component on an object | `onCreate`, `onUpdate`, `onInteract`, `onCollideWithPlayer`, `onButtonPress` and the rest of the [object events](events.md#object-callbacks) |
| Trigger script | **Script** field of a **Trigger box** component | `onTriggerEnter(index)`, `onTriggerExit(index)` |

A file is either `.lua` or `.blocks`; the extension decides which. **New Lua** and **New blocks** in the inspector create one in `scripts/` named after the object and open it. Press **Play** and the scene runs with your changes.

## Script environments

Each script runs in **its own environment**. Functions defined in one script are NOT automatically available to other scripts. To share functions (for example, from the scene script to object scripts), publish them to `_G`:

```lua
-- In the scene script
function addScore(amount)
    -- ...
end

-- Publish so object scripts can call it
_G.addScore = addScore
```

Then any object script can call `addScore(100)` directly, because `_G` is the fallback for all environments.

## In this section

1. [**Fixed-Point Math**](fixed-point.md): how numbers work on the PS1. Read this first.
2. [**Event Callbacks**](events.md): every callback and when it fires.
3. [**API Reference**](api-reference.md): every Lua function the engine provides.
4. [**Patterns**](patterns.md): working code for common game mechanics.
