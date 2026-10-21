# Lua API Reference

Every function the engine gives your Lua scripts, as of the current psxsplash source. Numbers that carry fractions are [fixed-point](fixed-point.md).

## Net

Linking consoles over the serial port (SIO1). See [Multiplayer](../editor/multiplayer.md) for how sessions, slots and the host work, and for running several consoles from the editor.

### Session

```lua
Net.Connect([baud [, rxMode]])  -- start a session -> true
Net.Disconnect()                -- leave it (tells the others if connected)
Net.IsConnected()               -- -> true once connected
Net.State()                     -- -> 0 not connected, 1 connecting, 2 connected,
                                --    3 version mismatch, 4 scene mismatch
Net.IsHost()                    -- -> true on the console that owns world objects
Net.LocalSlot()                 -- -> this console's slot (255: none yet)
Net.PlayerCount()               -- -> players in the session
Net.Stats()                     -- -> table of link counters
```
`Net.Connect` uses the scene's Network id to check that every console runs the same scene. `baud` defaults to 57600. `rxMode` is `0` Auto (the default), `1` Polled or `2` Interrupt.

`Net.Stats` fields include `bytesRx`, `bytesTx`, `frames`, `crcErrors`, `resyncs`, `serialErrors`, `rttMillis`, `goodput`, `queueDepth`, `baud`, `rxMode` and `heapKB`.

### Avatars and world objects

```lua
Net.SetLocalAvatar(actor)           -- the actor this console sends as its player
Net.SetRemoteAvatar(slot, actor)    -- the local actor that shows slot's player
Net.SetReplicationEnabled(enabled)  -- send this console's avatar at all (default true)
Net.RegisterActor(actor)            -- host owns this actor -> false if 48 are registered
Net.UnregisterActor(actor)
Net.SyncActor(actor)                -- send the object's self.sync table -> bool
```
Each takes an actor handle from [`Actor.Find`](#actor). The local avatar defaults to the player. Turn replication off in scenes with no avatar, such as a menu or a lobby. `Net.SyncActor` returns `false` if the object has no `self.sync` table or it does not fit in 254 bytes; the other consoles apply it only for a registered actor, and only when it comes from the host.

### Messages

```lua
Net.Send(eventId [, arg])   -- two integers, reliable -> bool
Net.SendData(str)           -- a string of 1 to 256 bytes, reliable and in order -> bool
Net.ReliableQueueDepth()    -- -> queued, capacity
```
The other consoles receive these in the scene script's `onNetEvent(id, arg)` and `onNetData(str)`. Both send functions return `false` when not connected or when the reliable queue is full, and the message is dropped; check the result, or compare `Net.ReliableQueueDepth()` before sending a batch.

### Across scene loads

```lua
Net.SetPersistent(enabled)  -- keep the session through the next Scene.Load
Net.IsPersistent()          -- -> bool
```
Call `Net.SetPersistent(true)` before `Scene.Load`, then `Net.Connect()` again in the new scene: the console keeps its slot. Avatar mappings and registered actors are still dropped with the old scene.

---

## Entity

Manage game objects in the scene.

### Finding Objects

```lua
Entity.Find(name)
```
Find an object by its name in the scene tree (string). Returns the object or `nil`.

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
    Like `SetPosition`, these move the transform directly - they ignore navigation regions and collision. They affect the object, not the [player](#player).

### Texture Manipulation

Change how an object's polygons are textured at runtime. Useful for scrolling textures, sprite-sheet animation, and swapping texture pages.

```lua
Entity.SetUVOffset(object, u, v)
```
Set an **absolute** UV offset (0-255 each) applied non-destructively at render time. The source polygon UVs are left untouched, so you can animate this freely every frame. This is the same value driven by the **UV Offset** [timeline track](../editor/timeline.md#tracks).

```lua
Entity.SetUVs(object, {u, v})
```
**Additively** shift the UVs of every polygon on the object by `(u, v)`. Unlike `SetUVOffset`, this mutates the stored polygon UVs, so repeated calls accumulate.

```lua
Entity.SetTPage(object, {x, y})
```
Set the texture page (VRAM page coordinates) for every polygon on the object - effectively swapping which region of VRAM it samples from.

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
Snap `child` to `parent`: the child is placed at the parent's position plus `offset` (a Vec3 expressed in the **parent's local space**) and inherits the parent's rotation. This is a one-shot reparent each call - to keep a child attached, call it every frame in `onUpdate`.

```lua
-- Keep a "held item" attached to a character's hand bone offset
function onUpdate(self, dt)
    local hand = Entity.Find("Character")
    Entity.SetParent(hand, self, Vec3.new(FixedPoint.new(0), FixedPoint.new(1), FixedPoint.new(0)))
end
```

The scene tree is flattened at export, so a parent in the editor does not carry its children along. `Entity.SetParent` is how you attach at runtime.

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
    State queries are **per-player**. Player 1 reads the controller in **port 1**, player 2 reads **port 2**. The old single-controller names (`Input.IsPressed`, `Input.IsReleased`, `Input.IsHeld`, `Input.GetAnalog`) were **removed** - use the `Player1` / `Player2` variants below. If you only support one player, use the `Player1` functions.

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
    The `onButtonPress` / `onButtonRelease` object callbacks fire for presses on **either** controller. They receive only the button id, not the player number - if you need to tell players apart, poll `Input.*Player1` / `Input.*Player2` directly (e.g. in `onUpdate`).

### Driving something other than the player

```lua
Input.BindToActor(player, actor)
```
Point a controller's built-in locomotion at any [actor](#actor). `player` is `1` or
`2`. The pad then walks and turns that actor with the engine's normal movement,
collision and camera handling, instead of the scene's Player.

```lua
Input.GetBoundActor(player)   -- -> actor id as a plain number
```

```lua
-- hand pad 1 back to the player
Input.BindToActor(1, Actor.GetPlayer())
```

Binding is how you build local co-op or possessable NPCs.

!!! warning "GetBoundActor returns an id, not a handle"
    It gives you the actor **index** as a number. Pass it through
    `Actor.FindByIndex` before calling any other `Actor.*` function on it.

---

## Camera

Control the scene camera.

```lua
Camera.FollowPsxPlayer(bool)
```
Sets if the camera should be controlled by the Player. Set false if you want to use the lua functions below to control the camera. 

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

### Following an actor

```lua
Camera.SetFollowTarget(actor)
```
Point the follow camera at any [actor](#actor) instead of the player. The camera
keeps its usual orbit behaviour; only the thing it orbits changes.

```lua
Camera.GetFollowTarget()   -- -> actor handle, or nil if following nothing
Camera.ClearFollowTarget() -- stop following; the camera holds its last transform
```

This is how you hand the camera to a vehicle or a spectator target.

!!! warning "Navigation controller override"
    In scenes with a Player and navigation regions, the navigation controller continuously overrides camera position and rotation. Manual camera changes via these functions will be overwritten on the next frame. The Camera API is primarily useful during cutscenes, which temporarily suspend the navigation controller.

```lua
Camera.LookAt(x, y, z)
Camera.LookAt(vec3)
```
Turn the camera toward a world point. It sets the camera's rotation once; call it each frame to keep tracking.

!!! tip "Check out the example script"
    "Lua Free Cam" in the patterns section uses these camera functions.

---

## Player

Control the player.

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
    "Player Position and Rotation" in [Patterns](patterns.md).

---

## UI

Canvas and element manipulation. Canvases and their elements are built in the [UI canvas editor](../editor/ui-canvas.md).

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

### Sheet Cell

An Image backed by a sprite sheet cell (set in the editor's **Sheet** field) can show another cell of the same sheet:

```lua
UI.SetFrame(handle, cell)                -- cell 0-255, row by row from the top left
UI.GetFrame(handle)                      -- Returns the cell, or -1 for a plain Image
```

`SetFrame` does nothing on an Image that is not backed by a sheet.

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

!!! warning "These are ANCHOR-relative, not screen coordinates"
    `x` and `y` are the element's offset from its anchor, which is what the
    splashpack stores; the runtime adds the anchor's own position when it draws.
    For a centre-anchored canvas on a 320x240 display that is a (160, 120)
    difference. `GetPosition` and `SetPosition` speak the same space, so reading
    a position and writing it back is exact - but passing a screen coordinate to
    `SetPosition` is not.

### Immediate-Mode Drawing

Draw primitives straight to the GPU this frame, without any pre-authored element. Coordinates are PS1 screen pixels (320x240); colors are RGB 0-255. Call these from `onUpdate` to keep them on screen - they are **not** persistent and must be re-issued every frame.

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

Play sound effects and music. Clips are set up with an [Audio clip component](../editor/audio.md).

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

Play music tracks burned onto the disc as CD-DA audio. CD-DA only works when running from a disc image, and the editor writes no CD-DA tracks into the images it makes (see [Audio](../editor/audio.md#cd-da)). Track numbers start at 2 because track 1 is the data track.

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

!!! warning "No CD-DA from the editor"
    CD-DA needs audio tracks on the disc. Play and Run on hardware load files over PCdrv, and the disc image the editor writes has one data track. Use [audio clips](../editor/audio.md) for music. A scene with [world streaming](../editor/world-streaming.md) on ignores `Audio.PlayCDDA` as well.

---

## Light

Control runtime point lights. Only lights marked runtime in the scene exist at runtime; baked lights are part of the vertex colours and cannot be found. See [Lighting](../editor/lighting.md).

```lua
Light.Find(name)
```
Find a light by its object name. Returns a light handle (a number), or `nil` if there is no such light.

```lua
Light.SetPosition(light, vec3)
```
Move a light. Takes a Vec3 table `{x, y, z}` in world coordinates, the same units as [`Entity.SetPosition`](#entity).

```lua
Light.SetColor(light, r, g, b)           -- RGB 0-255 (clamped)
Light.SetRadius(light, radius)           -- same units as positions; light fades to zero at this distance
Light.SetIntensity(light, intensity)     -- 1.0 = normal brightness
Light.SetEnabled(light, enabled)         -- true / false
```

```lua
-- A torch that follows an object
local torch = Light.Find("Torch")

function onUpdate(self, dt)
    Light.SetPosition(torch, Entity.GetPosition(self))
end
```

Light functions do nothing if the handle is not a valid light.

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
Returns the current scene's index (0-based, the position in the [build list](../editor/scene-settings.md#build-scenes)).

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

Read and write real saves on a physical (or emulated) PlayStation memory card. Unlike `Persist`, these survive power-off and appear in the BIOS memory card manager. See [Scene Settings](../editor/scene-settings.md#memory-card) for project setup (region/product code, save title, BIOS icon).

`port` is `0` (slot 1) or `1` (slot 2). **Every function returns two values**: a result and an error string. On success the error is `nil`; on failure the result is `false`/`nil` and the error is a human-readable message - nothing fails silently.

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
Write a fresh, empty Sony filesystem to the card. **Erases everything on the card** - only offer this behind an explicit "format card" confirmation.

---

## Cutscene

Control cutscene playback. Only one at a time. Cutscenes are built in the [timeline](../editor/timeline.md).

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

Control animation playback. Multiple simultaneous instances. Animations are built in the [timeline](../editor/timeline.md).

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

Control bone-based skinned mesh animations. See [Skinned Animation](../editor/skinned-animation.md).

Each skinned object can play one clip at a time. Clips are referenced by the **object name** (the object with the Skinned animation component) and the **clip name** (the `name` inside the `.anim` file, which is the animation's name in the model file).

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

### SkinnedAnim.GetBone

```lua
local pos, rot = SkinnedAnim.GetBone(objectName, bone)  -- bone: name or index
```

Returns where a bone is right now: its world position as a Vec3, and its world rotation as a Vec3 (the same convention as `Entity.GetRotation`). Returns `nil` if the object is not skinned or the bone does not exist. Use it to attach a weapon or an effect to a hand.

```lua
local hand = SkinnedAnim.GetBone("Hero", "hand_r")
if hand then Entity.SetPosition(Entity.Find("Sword"), hand) end
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

Use this during cutscenes, dialogue, or any time the player shouldn't move. Disabling controls only freezes built-in movement/look - button event callbacks still fire, so menus and dialogue advancement keep working.

```lua
-- Freeze everyone for a cutscene
Controls.SetEnabledPlayer1(false)
Controls.SetEnabledPlayer2(false)
```

!!! note "Built-in movement is shared"
    The engine's built-in player movement responds to **both** controller ports moving the **same** player (so a second pad acts as a co-pilot). These toggles enable/disable each port's contribution to that movement. For genuinely independent two-player gameplay, disable the built-in movement and drive a second character yourself from the [`Input.*Player2`](#input) functions plus [`Entity`](#entity) movement.

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
Project a 3D world position through the **current camera** to 2D screen coordinates (320x240 space). Returns two numbers. Useful for pinning UI or [immediate-mode lines](#ui) onto a moving 3D object - for example, drawing a marker over an enemy's head.

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

```lua
Debug.DrawLine(fromVec3, toVec3 [, colorVec3])
Debug.DrawBox(centerVec3, sizeVec3 [, colorVec3])
```

!!! warning "The draw calls are accepted but nothing is drawn"
    Both functions validate their arguments and return without queueing anything.
    They are safe to leave in a script - they simply have no visible effect yet.
    See [Known Issues](../reference/known-issues.md).

---

## Actor

An **actor** is a movable thing with a position and rotation: the player, or any
scene object you address by name. Actor handles are what `Input.BindToActor` and the camera follow functions take.

!!! warning "Actor handles are not Entity handles"
    `Entity.*` functions take an object handle from `Entity.Find`; `Actor.*`
    functions take an actor handle from `Actor.Find`. Calling `Entity.GetPosition`
    on an actor handle returns `nil` rather than raising, so the mistake shows up
    later as a nil-index error somewhere unrelated. Use `Actor.GetEntity(actor)`
    to cross over deliberately.

### Finding actors

```lua
Actor.GetPlayer()         -- the player's actor
Actor.Find(name)          -- by object name -> actor or nil
Actor.FindByIndex(index)  -- by actor id -> actor or nil
Actor.GetCount()          -- how many actors the scene has
Actor.IsPlayer(actor)     -- true if this is the player
Actor.GetName(actor)      -- object name -> string or nil
Actor.GetEntity(actor)    -- the matching Entity handle, for Entity.* calls
```

### Position and rotation

```lua
Actor.GetPosition(actor)          -- -> {x, y, z} of FixedPoint
Actor.SetPosition(actor, vec3)
Actor.GetRotation(actor)          -- -> {x, y, z}
Actor.SetRotation(actor, vec3)
```

```lua
local x, z = Actor.GetPositionXZ(actor)
```
The same position as **two plain integers**, in pixels, on the ground plane.

!!! tip "Prefer GetPositionXZ in per-frame code"
    `GetPosition` builds four Lua tables per call (the vector, plus a FixedPoint
    for each component). Called for every actor every frame that is thousands of
    short-lived tables a second for the collector to deal with on a 33MHz CPU.
    `GetPositionXZ` allocates nothing.

### Navigation

```lua
Actor.GetNavRegion(actor)         -- current nav region index, or nil
Actor.FindPath(actor, x, y, z)    -- path to a point
```

---

## Agent

Agents are objects with an **Agent (NPC)** component (see [Agents](../editor/agents.md)): the engine walks them along the nav regions, patrols their waypoints and checks what they see and hear. Every function takes an **actor** handle: `self` in the agent's own script, or `Actor.Find(name)`. Positions are `{x, y, z}` tables in engine units (scene metres divided by the GTE scaling), Y down, like `Actor.GetPosition`. The engine runs agents only in a scene with nav regions.

### Enabling and moving

```lua
Agent.IsAgent(actor)  -- -> bool
```
True if the object has an Agent component, or was moved by `MoveTo` or `SetTarget`.

```lua
Agent.SetEnabled(actor, bool)
```
Turns the agent on or off. Off stops it where it is and drops its target.

```lua
Agent.IsEnabled(actor)  -- -> bool
```
Whether the agent is running.

```lua
Agent.MoveTo(actor, vec3)
```
Walks to a position along the nav regions; pass another actor instead of a position to follow it. Enables the agent first. In the Patrol state the next waypoint is picked again whenever a move ends.

```lua
Agent.SetTarget(actor, target)
```
Follows another actor (`Actor.GetPlayer()` for the player); it is also who the agent watches and listens for. Enables the agent first.

```lua
Agent.GetTarget(actor)  -- -> actor or nil
```
The actor it follows, or nil.

```lua
Agent.Stop(actor)
```
Stops moving and drops the target.

```lua
Agent.IsMoving(actor)  -- -> bool
```
Whether it is walking somewhere.

```lua
Agent.SetSpeed(actor, speed)
```
Walking speed per frame, in engine units (a FixedPoint).

```lua
Agent.GetSpeed(actor)  -- -> number
```
Walking speed per frame as a whole number of engine units: see the warning below.

### State machine

```lua
Agent.GetState(actor)  -- -> state
```
The state number: 0 Idle, 1 Patrol, 2 Seek, 3 Flee, 4 Attack, 5 Wander, 6 Investigate, 7 Custom.

```lua
Agent.SetState(actor, state)
```
Changes the state: runs `onStateExit` and `onStateEnter` on the agent's script and plays the state's skin clip when one is set.

### Senses

```lua
Agent.CanSee(actor, target)  -- -> bool
```
True when `target` is inside the vision range and cone and within the region hops. Ranges are measured as the larger of the X and Z distances, so they are squares, not circles.

```lua
Agent.CanHear(actor, target)  -- -> bool
```
True when `target` is inside the hearing range (a square, like the vision range).

```lua
Agent.SetVisionRange(actor, range)
```
Vision range in engine units; 0 sees nothing.

```lua
Agent.GetVisionRange(actor)  -- -> number
```
Vision range as a whole number of engine units: see the warning below.

```lua
Agent.SetVisionAngle(actor, deg)
```
HALF the cone in degrees, 0 to 180 (the inspector's Field of view is the whole cone); 180 sees all around.

```lua
Agent.SetHearingRange(actor, range)
```
Hearing range in engine units; 0 hears nothing.

```lua
Agent.GetHearingRange(actor)  -- -> number
```
Hearing range as a whole number of engine units: see the warning below.

```lua
Agent.SetAlertTimeout(actor, frames)
```
Frames after it last sensed the target before `onTargetLost` runs.

```lua
Agent.GetLastKnownPos(actor)  -- -> {x, y, z} or nil
```
Where it last saw or heard its target, or nil.

!!! warning "The getters return whole numbers"
    `GetSpeed`, `GetVisionRange` and `GetHearingRange` return a plain number, the engine value with its fraction dropped: at the default GTE scaling an 8 m vision range (0.08 engine units) reads back as 0. Keep your own copy of a value you set.

### Patrol

```lua
Agent.AddWaypoint(actor, vec3)
```
Appends a patrol waypoint and turns patrolling on; the engine holds 8 and ignores more.

```lua
Agent.ClearWaypoints(actor)
```
Removes every waypoint and turns patrolling off.

```lua
Agent.SetPatrolEnabled(actor, bool)
```
Turns patrolling on or off; on, an Idle agent with waypoints switches to Patrol.

Agent events (`onStateEnter`, `onTargetSeen`, `onPatrolPoint`, ...) are listed under [Events](events.md#agent-callbacks).

---

## Sprite

2D pictures cut from the scene's [sprite sheets](../editor/sprites.md). Sheets and animations are authored in the editor; sprites are made here, at run time. A sprite is a plain number, its slot in a pool of **128**: keep it in a variable or a table. Functions given an id that is not a live sprite do nothing (`IsVisible` returns false).

!!! warning "Pass names, not numbers"
    `Sprite.Create`, `Sprite.PlayAnim` and `Sprite.SetFacingFromYaw` take a sheet or animation name. They also mean to take an index, but the engine checks for a string first and Lua counts a number as a string, so `Sprite.Create(0)` looks for a sheet **named** "0" and returns -1. Always pass the name.

### Sheets and animations

```lua
Sprite.SheetIndex(name)  -- -> index or -1
```
The sheet's index in the scene, or -1 when no sheet has that name.

```lua
Sprite.AnimIndex(name)  -- -> index or -1
```
The animation's index, or -1 when none has that name. It searches every sheet and returns the first match, which is why the editor makes animation names unique across the scene.

### Creating and destroying

```lua
Sprite.Create(sheetName)  -- -> id or -1
```
Makes a sprite from a sheet and returns its id, 0 to 127. Returns -1 when the sheet does not exist or all 128 are in use. A new sprite is visible, on the screen plane at (0, 0), on layer 0, with a neutral tint, showing cell 0 and no animation.

```lua
Sprite.Destroy(id)
```
Frees the slot. The id may be handed out again by the next `Create`.

```lua
Sprite.Count()  -- -> number
```
How many sprites exist.

### Position

```lua
Sprite.SetPos(id, x, y)
```
Screen position of the sprite's top left corner, in whole pixels (-32768 to 32767).

```lua
Sprite.BindToActor(id, actor, plane, offsetX, offsetY)
```
Makes the sprite follow an object every frame. `actor` is an `Actor.Find` or `Entity.Find` handle; `nil` unbinds it. `plane` is 0 for the screen plane (the default) or 1 for the world plane; the offsets default to 0. On the screen plane the sprite's x is the object's X and its y the object's **Z**, in whole engine units, plus the offsets; the object's height is ignored. The bound position replaces whatever `SetPos` set.

```lua
Sprite.SetWorldPos(id, x, y, z)
```
Position on the world plane. The engine keeps it but does **not draw** world-plane sprites yet.

```lua
Sprite.SetViewOffset(x, y)
Sprite.GetViewOffset()  -- -> x, y
```
A scroll for every sprite at once, in pixels: each one is drawn at its position minus this offset. Starts at 0, 0.

```lua
Sprite.SetIgnoreViewOffset(id, bool)
```
True pins the sprite to the screen, ignoring the view offset (for a score or a cursor).

### Animation

```lua
Sprite.PlayAnim(id, animName, restart)
```
Plays an animation; the sprite switches to the animation's sheet. If the same animation is already playing it carries on, so calling this every frame is fine; pass `true` as `restart` to start it again from its first cell. An unknown name does nothing. Each cell shows for the animation's frame time in vsyncs, 60 a second of real time. A looping animation starts again after its last cell; otherwise it stops there.

```lua
Sprite.StopAnim(id)
```
Stops the animation on the cell it is showing.

```lua
Sprite.SetFrame(id, cell)
```
Shows one cell of the sprite's sheet, 0 to 255, counted row by row from the top left, and drops the animation. The cell is not checked against the sheet's size.

```lua
Sprite.SetFacingFromYaw(id, firstAnimName, directions, yaw)
```
For a character drawn from several directions: plays animation `firstAnimName` + n, where n (0 to `directions` - 1) is the direction nearest to `yaw`. The directional animations must follow one another in the sheet's list, starting with yaw 0 and going the way yaw increases. `yaw` is in pi-units like `Entity.SetRotationY` (1 = 180 degrees); yaw 0 is an object facing the same way as the camera, so pass the object's yaw minus `Camera.GetRotation().y`. The editor's **Add direction set** names them in this order, starting with `_n`. Changing direction keeps the cell and timing, so a walk cycle does not restart, and keeps whether it is playing: call `PlayAnim` first. `directions` of 0 does nothing.

### Look

```lua
Sprite.SetVisible(id, bool)
Sprite.IsVisible(id)  -- -> bool
```
Hides or shows the sprite.

```lua
Sprite.SetFlip(id, flipX, flipY)
```
Mirrors the picture left to right and/or top to bottom.

```lua
Sprite.SetColor(id, r, g, b)
```
Tints the sprite, each 0 to 255. 128, 128, 128 is the picture's own colours; higher brightens, up to double.

```lua
Sprite.SetSize(id, w, h)
```
Draws the cell stretched to `w` x `h` pixels. 0 for either keeps the cell's own width or height.

```lua
Sprite.SetLayer(id, layer)
```
Drawing order, 0 (front) to 11; a higher number is drawn as 11. All sprites are drawn in front of the 3D scene and behind UI canvases.

!!! tip "Unscaled sprites are cheapest"
    A sprite at its cell size and not flipped is drawn as a single GPU sprite. Flipping or a size other than the cell's makes it a textured quad.

---

## Tile

Questions to the scene's [tilemap](../editor/tilemaps.md): what is walkable, and where its objects are. The map, its brushes and its objects are authored in the editor; Lua only reads them. Positions are whole pixels with the map's top left corner at 0, 0, x across the map and z down it: the same pixel space as the sprites, which a 2D game shares with the world X and Z of its actors. Off the map is not walkable.

Without a tilemap, `Tile.Walkable`, `Tile.RayClear` and `Tile.MoveActor` behave as if nothing were in the way, so movement written with them works in any scene.

### The map

```lua
Tile.Active()  -- -> bool
```
True when the scene has a tilemap.

```lua
Tile.MapSize()  -- -> width, height, tileW, tileH
```
The map's size in tiles and the tile size in pixels. All four are 0 without a map.

### Walls

```lua
Tile.Walkable(x, z)  -- -> bool
```
Whether the cell under pixel x, z is walkable. False off the map and on empty or hole cells; true when the scene has no tilemap.

```lua
Tile.RayClear(x0, z0, x1, z1)  -- -> bool
```
Whether the straight line between two pixels crosses only walkable cells, for line of sight. The first pixel's own cell is not tested; the last one's is, so a point inside a wall is not visible. The line is sampled every half tile (half the smaller tile side) and gives up after 128 samples: a longer line returns false. True without a tilemap.

```lua
Tile.MoveActor(actor, dx, dz, ignoreWalls)  -- -> x, z
```
Moves an actor (an `Actor.Find` or `Actor.GetPlayer` handle) by `dx`, `dz` pixels on its world X and Z, and returns where it ends up, in whole units. X is tried first, then Z, each on its own: a wall stops the move on that axis, so the actor slides along it. The actor's height is left alone. Pass `true` as `ignoreWalls` to move without looking at the map; without a tilemap the whole move is always made. Given something that is not an actor it returns a single null userdata, which is not `nil`.

### Objects

```lua
Tile.ObjectCount()  -- -> number
```
How many objects the map has, at most 128: one for each cell painted with a brush that has a kind, in reading order. 0 without a map.

```lua
Tile.ObjectAt(i)  -- -> kind, id, x, z
```
Object `i`, counted **from 1**: its kind, its id and the centre of its tile in pixels. Kinds and ids are the numbers set on the brush in the editor; the engine gives them no meaning.

!!! warning "Do not ask past the count"
    For an index outside 1 to `Tile.ObjectCount()`, `ObjectAt` returns a single value that is **not** `nil` (a null userdata), so `if Tile.ObjectAt(i) then` does not find the end. Loop with `for i = 1, Tile.ObjectCount() do`.
