# Lua API Reference

Complete reference for every function available in SplashEdit Lua scripts.

---

## Entity

Manage game objects in the scene.

### Finding Objects

```lua
Entity.Find(name)
```
Find an object by its GameObject name (string). Returns the object or `nil`.

```lua
Entity.FindByIndex(index)
```
Find an object by its array index (0-based). Returns the object or `nil`.

```lua
Entity.FindByScriptIndex(index)
```
Find the first object with a matching Lua script file index. Returns the object or `nil`.

```lua
Entity.GetCount()
```
Returns the total number of objects in the scene.

```lua
Entity.ForEach(callback)
```
Iterate all active objects. The callback receives `(object, index)` for each. Skips inactive objects.

```lua
Entity.ForEach(function(obj, i)
    Debug.Log("Object " .. i .. " is active")
end)
```

### Active State

```lua
Entity.SetActive(object, bool)
```
Show or hide an object. Fires `onEnable` or `onDisable` callbacks.

```lua
Entity.IsActive(object)
```
Returns `true` if the object is active.

### Position

```lua
Entity.GetPosition(object)
```
Returns a Vec3 table `{x, y, z}` in world coordinates.

```lua
Entity.SetPosition(object, vec3)
```
Set position instantly (no physics, no interpolation).

```lua
-- Example
local pos = Entity.GetPosition(self)
Entity.SetPosition(self, Vec3.new(pos.x + 1, pos.y, pos.z))
```

### Rotation

```lua
Entity.GetRotationY(object)
```
Get the Y-axis rotation as an angle in pi-units.

```lua
Entity.SetRotationY(object, angle)
```
Set the Y-axis rotation. Angle is in pi-units (1 = 180 degrees, `FixedPoint.new(1) / 2` = 90 degrees).

```lua
Entity.SetRotation(object, vec3)
```
Set the full rotation from a Vec3 of Euler angles `{x, y, z}` in pi-units. Applied as a Y * X * Z rotation matrix (same convention as `Camera.SetRotation`).

### Direction Vectors

Get an object's local axes in world space, derived from its current rotation. Each returns a Vec3 `{x, y, z}`.

```lua
Entity.GetForward(object)   -- local +Z in world space
Entity.GetRight(object)     -- local +X in world space
Entity.GetUp(object)        -- local +Y in world space
```

```lua
-- Spawn something 2 units in front of an object
local fwd = Entity.GetForward(self)
local pos = Entity.GetPosition(self)
local spawnPos = Vec3.add(pos, Vec3.mul(fwd, FixedPoint.new(2)))
```

### Local Movement

Move an object along its **own** local axes (relative to its rotation), by a fixed-point `step`. These are convenience wrappers over `GetForward`/`GetRight`/`GetUp` + `SetPosition`.

```lua
Entity.MoveForward(object, step)
Entity.MoveBackward(object, step)
Entity.MoveLeft(object, step)
Entity.MoveRight(object, step)
Entity.MoveUp(object, step)
Entity.MoveDown(object, step)
```

```lua
-- Drive an object forward along its facing each frame
function onUpdate(self, dt)
    Entity.MoveForward(self, FixedPoint.new(1) / 32)
end
```

!!! note "No collision"
    Like `SetPosition`, these move the transform directly — they ignore navigation regions and collision. They affect the object, not the [PsxPlayer](#player).

### Texture Manipulation

Change how an object's polygons are textured at runtime. Useful for scrolling textures, sprite-sheet animation, and swapping texture pages. See [UV Offsetting](../components/objects.md#uv-offset-animation) for the full workflow.

```lua
Entity.SetUVOffset(object, u, v)
```
Set an **absolute** UV offset (0-255 each) applied non-destructively at render time. The source polygon UVs are left untouched, so you can animate this freely every frame. This is the same value driven by the `Object UV Offset` [timeline track](../components/animations.md#track-types).

```lua
Entity.SetUVs(object, {u, v})
```
**Additively** shift the UVs of every polygon on the object by `(u, v)`. Unlike `SetUVOffset`, this mutates the stored polygon UVs, so repeated calls accumulate.

```lua
Entity.SetTPage(object, {x, y})
```
Set the texture page (VRAM page coordinates) for every polygon on the object — effectively swapping which region of VRAM it samples from.

```lua
-- Scroll a texture horizontally (e.g. a waterfall or conveyor belt)
local scroll = 0
function onUpdate(self, dt)
    scroll = (scroll + 1) % 256
    Entity.SetUVOffset(self, scroll, 0)
end
```

### Parenting

```lua
Entity.SetParent(parent, child, offset)
```
Snap `child` to `parent`: the child is placed at the parent's position plus `offset` (a Vec3 expressed in the **parent's local space**) and inherits the parent's rotation. This is a one-shot reparent each call — to keep a child attached, call it every frame in `onUpdate`.

```lua
-- Keep a "held item" attached to a character's hand bone offset
function onUpdate(self, dt)
    local hand = Entity.Find("Character")
    Entity.SetParent(hand, self, Vec3.new(FixedPoint.new(0), FixedPoint.new(1), FixedPoint.new(0)))
end
```

### Self Properties

Object scripts have shorthand access via `self`:

```lua
self.position      -- {x, y, z} (read/write)
self.active        -- boolean (read/write)
self.rotation      -- {x, y, z} (read/write)
self.rotationY     -- angle in pi-units (read/write)
```

---

## Vec3

3D vector math. All functions return new tables.

```lua
Vec3.new(x, y, z)          -- Create vector
Vec3.add(a, b)              -- a + b
Vec3.sub(a, b)              -- a - b
Vec3.mul(v, scalar)         -- v * scalar
Vec3.dot(a, b)              -- Dot product (scalar)
Vec3.cross(a, b)            -- Cross product (vector)
Vec3.length(v)              -- Magnitude ||v||
Vec3.lengthSq(v)            -- Squared magnitude (faster, no sqrt)
Vec3.normalize(v)           -- Unit vector (length = 1)
Vec3.distance(a, b)         -- Distance between two points
Vec3.distanceSq(a, b)       -- Squared distance (faster)
Vec3.lerp(a, b, t)          -- Linear interpolation: a + (b-a)*t
```

### Examples

```lua
local a = Vec3.new(1, 0, 0)
local b = Vec3.new(0, 1, 0)

local sum = Vec3.add(a, b)           -- {1, 1, 0}
local cross = Vec3.cross(a, b)       -- {0, 0, 1}
local dot = Vec3.dot(a, b)           -- 0
local one = FixedPoint.new(1)
local mid = Vec3.lerp(a, b, one / 2) -- {0.5, 0.5, 0}
local dist = Vec3.distance(a, b)     -- ~1.41
```

!!! tip "Use squared versions for comparisons"
    `Vec3.distanceSq` and `Vec3.lengthSq` avoid a square root. If you're comparing distances (e.g., "is the player within range?"), compare squared distances instead.

---

## Input

Controller button constants and state queries.

### Button Constants

```lua
Input.CROSS          Input.CIRCLE         Input.SQUARE         Input.TRIANGLE
Input.L1             Input.L2             Input.R1             Input.R2
Input.START          Input.SELECT
Input.UP             Input.DOWN           Input.LEFT           Input.RIGHT
Input.L3             Input.R3
```

### State Queries

!!! warning "Two-player API change"
    State queries are **per-player**. Player 1 reads the controller in **port 1**, player 2 reads **port 2**. The old single-controller names (`Input.IsPressed`, `Input.IsReleased`, `Input.IsHeld`, `Input.GetAnalog`) were **removed** — use the `Player1` / `Player2` variants below. If you only support one player, use the `Player1` functions.

```lua
Input.IsPressedPlayer1(button)
Input.IsPressedPlayer2(button)
```
Returns `true` **only on the frame** the button was pressed (edge trigger).

```lua
Input.IsReleasedPlayer1(button)
Input.IsReleasedPlayer2(button)
```
Returns `true` **only on the frame** the button was released.

```lua
Input.IsHeldPlayer1(button)
Input.IsHeldPlayer2(button)
```
Returns `true` **every frame** while the button is held down.

```lua
Input.GetAnalogPlayer1(stick)
Input.GetAnalogPlayer2(stick)
```
Read analog stick position. `stick` 0 = left stick, 1 = right stick. Returns two values: `x, y` in range -127 to +127 (0 = centered).

```lua
-- Player 1, left stick
local x, y = Input.GetAnalogPlayer1(0)

-- Simple two-player check
if Input.IsPressedPlayer1(Input.CROSS) then jump(1) end
if Input.IsPressedPlayer2(Input.CROSS) then jump(2) end
```

!!! note "Button events fire for both players"
    The `onButtonPress` / `onButtonRelease` object callbacks fire for presses on **either** controller. They receive only the button id, not the player number — if you need to tell players apart, poll `Input.*Player1` / `Input.*Player2` directly (e.g. in `onUpdate`).

---

## Camera

Control the scene camera.

```lua
Camera.FollowPsxPlayer(bool)
```
Sets if the camera should be controlled by the PsxPlayer. Set false if you want to use the lua functions below to control the camera. 

```lua
Camera.GetPosition()
```
Returns the camera's world position as vec3 `{x, y, z}`.

```lua
Camera.SetPosition(x, y, z)
Camera.SetPosition(vec3) -- {x, y, z}
```
Set camera position. Accepts three numbers or a vec3 table.

```lua
Camera.GetRotation()
```
Returns the camera's world rotation as Vec3 `{x, y, z}`

```lua
Camera.SetRotation(vec3)
```
Set camera rotation in pi-units. Applied as Y * X * Z rotation matrix. Accepts Vec3

```lua
Camera.MoveForward(stepAmount)
```
Moves the camera forward based on it's rotation and a fixed point number stepAmount

```lua
Camera.MoveBackward(stepAmount)
```
Moves the camera backward based on it's rotation and a fixed point number stepAmount

```lua
Camera.MoveLeft(stepAmount)
```
Moves the camera left based on it's rotation and a fixed point number stepAmount

```lua
Camera.MoveRight(stepAmount)
```
Moves the camera right based on it's rotation and a fixed point number stepAmount

```lua
Camera.GetForward()
```
Returns the camera's forward vector as Vec3 `{x, y, z}`

```lua
Camera.GetH()
```
Returns the current projection plane distance (H register). Default is 120. Higher values = narrower FOV (more telephoto), lower = wider FOV.

```lua
Camera.SetH(h)
```
Set the projection H register. Clamped to 1-1024. Use this to change the field of view at runtime.

The relationship between H and vertical FOV is: $\text{vFOV} = 2 \cdot \arctan\left(\frac{120}{H}\right)$

| H | Approx. Vertical FOV |
|---|-----|
| 60 | ~127° (ultra wide) |
| 120 | ~90° (default) |
| 200 | ~62° |
| 400 | ~33° (telephoto) |

!!! warning "Navigation controller override"
    In scenes with a PSXPlayer and navigation regions, the navigation controller continuously overrides camera position and rotation. Manual camera changes via these functions will be overwritten on the next frame. The Camera API is primarily useful during cutscenes, which temporarily suspend the navigation controller.

!!! warning "Incomplete functions"
    `Camera.LookAt()` exists but is a placeholder - it does not correctly point the camera at the target.

!!! tip "Check out the example script"
    "Lua Free Cam" in the patterns section uses these camera functions.

---

## Player

Control the PsxPlayer

```lua
Player.GetPosition()
```
Returns the player's position as Vec3 `{x, y, z}`.

```lua
Player.SetPosition(x, y, z)
Player.SetPosition(Vec3) -- {x, y, z}
```
Set player position. Accepts three numbers or a Vec3 table.

```lua
Player.GetRotation()
```
Returns the player's rotation as Vec3 `{x, y, z}`

```lua
Player.SetRotation(Vec3)
```
Set players rotation. Accepts Vec3

!!! tip "Check out the example script"
    "PsxPlayer Position and Rotation" in the patterns section.

---

## UI

Canvas and element manipulation. See [UI System](../components/ui.md) for setup.

### Canvas Operations

```lua
UI.FindCanvas(name)                      -- Find canvas by name -> index or -1
UI.SetCanvasVisible(nameOrIndex, bool)   -- Show/hide (accepts name string or index number)
UI.IsCanvasVisible(nameOrIndex)          -- Check visibility -> boolean
```

### Element Lookup

```lua
UI.FindElement(canvasIndex, name)        -- Find element by name -> handle or -1
UI.GetElementCount(canvasIndex)          -- Number of elements in canvas
UI.GetElementByIndex(canvasIndex, i)     -- Get element by position -> handle
UI.GetElementType(handle)               -- 0=Image, 1=Box, 2=Text, 3=Progress, 4=Line
```

### Visibility

```lua
UI.SetVisible(handle, bool)
UI.IsVisible(handle)
```

### Text

```lua
UI.SetText(handle, text)                -- Max 63 characters
UI.GetText(handle)                      -- Returns string
```

### Progress Bar

```lua
UI.SetProgress(handle, value)            -- 0-100 (clamped)
UI.GetProgress(handle)                   -- Returns number
```

### Color

```lua
UI.SetColor(handle, r, g, b)            -- RGB 0-255
UI.GetColor(handle)                     -- Returns r, g, b (three values)
UI.SetProgressColors(handle, bgR, bgG, bgB, fillR, fillG, fillB)
```

### Position & Size

```lua
UI.SetPosition(handle, x, y)
UI.GetPosition(handle)                   -- Returns x, y
UI.SetSize(handle, w, h)
UI.GetSize(handle)                       -- Returns w, h
```

### Immediate-Mode Drawing

Draw primitives straight to the GPU this frame, without any pre-authored element. Coordinates are PS1 screen pixels (320x240); colors are RGB 0-255. Call these from `onUpdate` to keep them on screen — they are **not** persistent and must be re-issued every frame.

```lua
UI.DrawLine(p1, p2, color)
```
Draw a single-pixel gouraud line from `p1` to `p2`. Each argument is a 2- or 3-element array table: `{x, y}` for the points, `{r, g, b}` for the color.

```lua
UI.DrawTriangle(p1, p2, p3, c1, c2, c3)
```
Draw a filled gouraud triangle. `p1`/`p2`/`p3` are `{x, y}` points; `c1`/`c2`/`c3` are per-vertex `{r, g, b}` colors (the fill is smoothly interpolated between them).

```lua
function onUpdate(self, dt)
    -- A red line across the screen
    UI.DrawLine({10, 120}, {310, 120}, {255, 0, 0})

    -- A triangle with a different color at each corner
    UI.DrawTriangle(
        {160, 40}, {120, 120}, {200, 120},
        {255, 0, 0}, {0, 255, 0}, {0, 0, 255})
end
```

!!! note "Drawn on top, every frame"
    These primitives are sent immediately and are not depth-sorted with your scene. Because they don't persist, you must call them each frame you want them visible. Pair with [`PSXMath.Convert3DTo2D`](#psxmath) to anchor them to world objects.

---

## Audio

Play sound effects and music. See [Audio](../components/audio.md) for setup.

```lua
Audio.Play(nameOrIndex, volume, pan)
```
Play a clip. Returns channel number (0-23) or -1 if all voices busy.

- `nameOrIndex`: clip name (string) or index (number)
- `volume`: 0-128 (default 100)
- `pan`: 0-127, where 0=left, 64=center, 127=right (default 64)

```lua
Audio.Find(name)         -- Find clip by name -> index or nil
Audio.Stop(channel)      -- Stop a specific channel
Audio.SetVolume(channel, volume, pan)  -- Adjust mid-playback
Audio.StopAll()          -- Stop all channels
```

### CD-DA Playback

Play music tracks burned onto the disc as CD-DA audio. CD-DA only works when running from a disc image (ISO build). Track numbers start at 2 because track 1 is the data track.

```lua
Audio.PlayCDDA(trackNo)
```
Start playing a CD-DA audio track by track number.

```lua
Audio.PauseCDDA()
```
Pause the currently playing CD-DA track.

```lua
Audio.ResumeCDDA()
```
Resume a paused CD-DA track.

```lua
Audio.StopCDDA()
```
Stop CD-DA playback entirely.

```lua
Audio.TellCDDA(callback)
```
Query the current playback position. The result is returned asynchronously through a callback function. The callback receives a single fixed-point value representing the playback position in seconds.

```lua
Audio.TellCDDA(function(position)
    Debug.Log("CD-DA position: " .. position)
end)
```

```lua
Audio.SetCDDAVolume(left, right)
```
Set the CD-DA output volume for left and right channels independently.

!!! warning "ISO builds only"
    CD-DA audio requires a disc image. It will not work when running via PCdrv (emulator or real hardware targets). Use the ISO build target to include CD-DA tracks.

---

## Scene

Multi-scene management.

```lua
Scene.Load(sceneIndex)
```
Request a scene transition. The load is **deferred to end-of-frame** - code after this call still executes.

```lua
Scene.GetIndex()
```
Returns the current scene's index (matches the order in the Control Panel Scenes tab, 0-based).

---

## Persist

Cross-scene persistent data storage. Survives scene loads but is lost on power-off.

```lua
Persist.Set(key, value)   -- Store a number with a string key
Persist.Get(key)          -- Retrieve -> number or nil if not set
```

!!! warning "Limits"
    Maximum **16 key-value pairs**. Values are numbers only. Silently fails if the table is full.

!!! tip "RAM vs. memory card"
    `Persist` keeps data only until power-off. To write a real save that survives a reboot and shows up in the BIOS, use [`MemCard`](#memcard).

---

## MemCard

Read and write real saves on a physical (or emulated) PlayStation memory card. Unlike `Persist`, these survive power-off and appear in the BIOS memory card manager. See the [Memory Cards guide](../components/memory-cards.md) for project setup (region/product code, save title, BIOS icon).

`port` is `0` (slot 1) or `1` (slot 2). **Every function returns two values**: a result and an error string. On success the error is `nil`; on failure the result is `false`/`nil` and the error is a human-readable message — nothing fails silently.

!!! warning "Blocking operations"
    Saving, loading, formatting and listing **block** for the duration of the card access (tens of milliseconds, occasionally more). Do them at deliberate save points, never every frame.

### MemCard.IsPresent

```lua
local present, err = MemCard.IsPresent(0)
```
Returns whether a card is inserted in the given port. (Absence is reported via `present == false`, not as an error.)

### MemCard.Save

```lua
local ok, err = MemCard.Save(port, key, table)
local ok, err = MemCard.Save(port, key, table, title)
```
Serialize a Lua table and write it as a save named `key`. Optional `title` overrides the project's configured BIOS title for this save.

Supported value types inside the table: `nil`, booleans, integers, strings, **nested tables**, and `FixedPoint` values. Functions, threads and other userdata cause an error. Tables may nest up to 16 levels deep (cycles are rejected).

```lua
local ok, err = MemCard.Save(0, "slot1", {
    level = 3,
    hp = FixedPoint.new(100),
    name = "HERO",
    flags = { door1 = true, bossDead = false },
})
if not ok then Debug.Log("Save failed: " .. err) end
```

### MemCard.Load

```lua
local data, err = MemCard.Load(port, key)
```
Load and deserialize a save. Returns the reconstructed table (with the same value types you saved) or `nil` plus an error if the save is missing or corrupt.

```lua
local data, err = MemCard.Load(0, "slot1")
if data then
    Persist.Set("level", Convert.FpToInt(data.hp))  -- example
    Debug.Log("Loaded " .. data.name)
end
```

### MemCard.Delete

```lua
local ok, err = MemCard.Delete(port, key)
```
Delete a save by key.

### MemCard.List

```lua
local names, err = MemCard.List(port)
```
Return an array of file names on the card (up to 15). Useful for showing existing saves or checking whether a slot is taken.

### MemCard.FreeBlocks

```lua
local blocks, err = MemCard.FreeBlocks(port)
```
Return the number of free 8 KiB blocks on the card (a standard card has 15). Check this before saving to a near-full card.

### MemCard.Format

```lua
local ok, err = MemCard.Format(port)
```
Write a fresh, empty Sony filesystem to the card. **Erases everything on the card** — only offer this behind an explicit "format card" confirmation.

---

## Cutscene

Control cutscene playback. Only one at a time. See [Cutscenes](../components/cutscenes.md).

```lua
Cutscene.Play(name)
Cutscene.Play(name, {
    loop = true,               -- Loop the cutscene
    onComplete = function()    -- Called when cutscene ends (not per-loop)
        -- ...
    end
})
Cutscene.Stop()                -- Stop current cutscene
Cutscene.IsPlaying()           -- Check if any cutscene is playing -> boolean
```

---

## Animation

Control animation playback. Multiple simultaneous instances. See [Animations](../components/animations.md).

```lua
Animation.Play(name)
Animation.Play(name, {
    loop = true,
    onComplete = function()
        -- ...
    end
})
Animation.Stop(name)           -- Stop all instances of named animation
Animation.Stop()               -- Stop ALL animations
Animation.IsPlaying(name)     -- True if any instance is active -> boolean
```

---

## SkinnedAnim

Control bone-based skinned mesh animations. See [Skinned Meshes](../components/skinned-meshes.md).

Each skinned object can play one clip at a time. Clips are referenced by the **object name** (the GameObject with a `PSXSkinnedObjectExporter`) and the **clip name** (the Unity `AnimationClip` asset name).

### SkinnedAnim.Play

```lua
SkinnedAnim.Play(objectName, clipName)
SkinnedAnim.Play(objectName, clipName, {
    loop = true,                -- Loop the animation (default: false)
    onComplete = function()     -- Called when a non-looping clip finishes
        -- ...
    end
})
```

Start playing a skinned animation clip. If the object is already playing, the new clip replaces it immediately.

```lua
-- Play idle loop
SkinnedAnim.Play("MyCharacter", "idle", { loop = true })

-- Play attack, then return to idle
SkinnedAnim.Play("MyCharacter", "attack", {
    onComplete = function()
        SkinnedAnim.Play("MyCharacter", "idle", { loop = true })
    end
})
```

### SkinnedAnim.Stop

```lua
SkinnedAnim.Stop(objectName)
```

Stop the skinned animation on the given object. The mesh freezes at its current pose. Also releases any `onComplete` callback.

### SkinnedAnim.IsPlaying

```lua
SkinnedAnim.IsPlaying(objectName) -- -> boolean
```

Returns `true` if the named skinned object is currently playing an animation.

### SkinnedAnim.GetClip

```lua
SkinnedAnim.GetClip(objectName) -- -> string or nil
```

Returns the name of the currently active clip, or `nil` if the object is not playing or not found.

```lua
local clip = SkinnedAnim.GetClip("MyCharacter")
if clip == "idle" then
    SkinnedAnim.Play("MyCharacter", "walk", { loop = true })
end
```

---

## Controls

Enable/disable player movement input, per player.

!!! warning "Two-player API change"
    These are now **per-player**, matching the [Input](#input) split. The old `Controls.SetEnabled` / `Controls.IsEnabled` names were **removed**. For a single-player game use the `Player1` variants.

```lua
Controls.SetEnabledPlayer1(bool)   -- true = player 1 can move, false = frozen
Controls.SetEnabledPlayer2(bool)   -- player 2 (controller port 2)
Controls.IsEnabledPlayer1()        -- Check state -> boolean
Controls.IsEnabledPlayer2()        -- Check state -> boolean
```

Use this during cutscenes, dialogue, or any time the player shouldn't move. Disabling controls only freezes built-in movement/look — button event callbacks still fire, so menus and dialogue advancement keep working.

```lua
-- Freeze everyone for a cutscene
Controls.SetEnabledPlayer1(false)
Controls.SetEnabledPlayer2(false)
```

!!! note "Built-in movement is shared"
    The engine's built-in PsxPlayer movement responds to **both** controller ports moving the **same** player (so a second pad acts as a co-pilot). These toggles enable/disable each port's contribution to that movement. For genuinely independent two-player gameplay, disable the built-in movement and drive a second character yourself from the [`Input.*Player2`](#input) functions plus [`Entity`](#entity) movement.

---

## Interact

Enable/disable interaction on specific objects.

```lua
Interact.SetEnabled(object, bool)    -- Enable/disable + hide prompt
Interact.IsEnabled(object)           -- Check state -> boolean
```

---

## Timer

Frame counting.

```lua
Timer.GetFrameCount()
```
Returns the number of frames since the current scene loaded. Resets to 0 on each scene load.

---

## PSXMath

Integer math utilities.

```lua
PSXMath.Clamp(value, min, max)     -- Clamp to range
PSXMath.Lerp(a, b, t)             -- Linear interpolation: a + (b-a)*t
PSXMath.Sign(value)                -- Returns -1, 0, or 1
PSXMath.Abs(value)                 -- Absolute value
PSXMath.Min(a, b)                  -- Minimum
PSXMath.Max(a, b)                  -- Maximum
```

All operate on fixed-point numbers. Use `FixedPoint.new(1) / 2` for fractional arguments like `t` in Lerp.

### Trigonometry

```lua
PSXMath.Cos(degrees)               -- Cosine, angle in DEGREES (0-360)
PSXMath.Sin(degrees)               -- Sine, angle in DEGREES (0-360)
```

These take the angle in **degrees** (a plain integer, not pi-units) and return a fixed-point value using the PS1's hardware trig tables. Convenient for circular/oscillating motion:

```lua
-- Bob an object up and down
function onUpdate(self, dt)
    local t = Timer.GetFrameCount() * 4   -- degrees per frame
    local pos = Entity.GetPosition(self)
    Entity.SetPosition(self, Vec3.new(pos.x, PSXMath.Sin(t % 360), pos.z))
end
```

### Projection

```lua
PSXMath.Convert3DTo2D(vec3)        -- Returns screenX, screenY (two numbers)
```
Project a 3D world position through the **current camera** to 2D screen coordinates (320x240 space). Returns two numbers. Useful for pinning UI or [immediate-mode lines](#ui) onto a moving 3D object — for example, drawing a marker over an enemy's head.

```lua
local enemy = Entity.Find("Enemy")
local p = Entity.GetPosition(enemy)
local sx, sy = PSXMath.Convert3DTo2D(p)
UI.DrawLine({sx - 4, sy}, {sx + 4, sy}, {255, 0, 0})
```

---

## Random

Accepts integers and generates random numbers for simulating dice, decks of cards, luck mechanics, etc. The functions Random.Number and Random.Range are generated based on the time. If you want to get the same sequence of numbers every time based on a seed use Random.GeneratorSeed, Random.GeneratorNumber, and Random.GeneratorRange.

```lua
Random.Number(max) 
```
Returns from 1 to max inclusive.
    
```lua
Random.Range(min,max) 
```
Returns from min inclusive to max inclusive 

```lua
Random.GeneratorSeed(newSeed) 
```
Sets the seed for the random number generator. 

```lua
Random.GeneratorNumber(max) 
```    
Returns from 1 to max inclusive

```lua
Random.GeneratorRange(min,max)
```
Returns from min inclusive to max inclusive

---

## FixedPoint

Create fixed-point numbers explicitly.

```lua
FixedPoint.new(integer)
```
Creates a fixed-point value from an integer. `FixedPoint.new(1)` = 1.0.

Useful for creating precise step sizes:

```lua
local one = FixedPoint.new(1)
local step = one / 64   -- Very small step (~0.015)
```

---

## Convert

Helper functions for working with fixed-point numbers. The raw integer of fixed point 1 equals 4096. 

```lua
Convert.FpToInt(fixedPoint)
```

Returns the raw integer representation of the passed fixed point value.

```lua
Convert.IntToFp(integer);
```

Returns the fixed point representation of the passed integer.

#### Expected Conversions
| Fixed Point | Raw Int |
|---|-----|
| 1.0 | 4096 |
| 0.5 | 2048 |
| 0.25 | 1024 |


---

## Debug

Development tools.

```lua
Debug.Log(message)
```
Print to the console. Visible in PCSX-Redux stdout when running on emulator.
