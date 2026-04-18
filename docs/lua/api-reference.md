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

```lua
Input.IsPressed(button)
```
Returns `true` **only on the frame** the button was pressed (edge trigger). Use in `onButtonPress` callbacks.

```lua
Input.IsReleased(button)
```
Returns `true` **only on the frame** the button was released.

```lua
Input.IsHeld(button)
```
Returns `true` **every frame** while the button is held down.

```lua
Input.GetAnalog(stick)
```
Read analog stick position. `stick` 0 = left stick, 1 = right stick. Returns two values: `x, y` in range -127 to +127 (0 = centered).

```lua
local x, y = Input.GetAnalog(0)  -- left stick
```

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
UI.GetElementType(handle)               -- 0=Image, 1=Box, 2=Text, 3=Progress
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

Enable/disable player input globally.

```lua
Controls.SetEnabled(bool)      -- true = player can move, false = frozen
Controls.IsEnabled()           -- Check state -> boolean
```

Use this during cutscenes, dialogue, or any time the player shouldn't move.

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
