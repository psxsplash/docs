# Patterns & Best Practices

Working code patterns for common game mechanics, with tested examples.

---

## Core Pattern: Event-Driven Design

SplashEdit games work best with fully event-driven architecture:

- `onInteract` for player-initiated actions
- `onButtonPress` for button-driven controls
- `onCollideWithPlayer` for collision responses
- `onTriggerEnter` / `onTriggerExit` for area triggers
- `Animation.Play` with callbacks for time-based actions
- `Cutscene.Play` with callbacks for sequenced events

---

## Core Pattern: Scene as Controller

The scene script is the central coordinator. Define shared functions there and publish them to `_G` so object scripts can access them:

```lua
-- scene.lua

local score = 0
local scoreText = -1

function onSceneCreationEnd()
    local hud = UI.FindCanvas("HUD")
    scoreText = UI.FindElement(hud, "ScoreText")
end

function addScore(amount)
    score = score + amount
    if scoreText >= 0 then
        UI.SetText(scoreText, "Score: " .. score)
    end
end

function setStatus(msg)
    Debug.Log("[Status] " .. msg)
end

-- CRITICAL: Publish to _G so object scripts can call these
_G.addScore = addScore
_G.setStatus = setStatus
```

Object scripts call these directly:

```lua
-- collectible.lua
function onCollideWithPlayer(self)
    addScore(100)
    setStatus("Got it!")
end
```

---

## Pattern: One-Time Action

Use a local flag to prevent repeated triggers:

```lua
local collected = false

function onCollideWithPlayer(self)
    if collected then return end
    collected = true

    Audio.Play("collect", 127, 64)
    Entity.SetActive(self, false)
    addScore(100)
    setStatus("Collected! +100 points")
end
```

---

## Pattern: Animation Gating

Prevent overlapping animations by checking `IsPlaying` before starting:

```lua
local isOpen = false

function onInteract(self)
    -- Don't start if either animation is running
    if Animation.IsPlaying("door_open") or Animation.IsPlaying("door_close") then
        return
    end

    -- Disable interaction during animation
    Interact.SetEnabled(self, false)

    if isOpen then
        isOpen = false
        Audio.Play("door_close", 100, 64)
        Animation.Play("door_close", {
            onComplete = function()
                Interact.SetEnabled(self, true)
            end
        })
    else
        isOpen = true
        Audio.Play("door_open", 100, 64)
        Animation.Play("door_open", {
            onComplete = function()
                Interact.SetEnabled(self, true)
            end
        })
    end
end
```

---

## Pattern: Dialogue System

Build a dialogue system in the scene script, then call it from NPC scripts:

### Scene Script (Dialogue System)

```lua
-- scene.lua
local dialogueCanvas = -1
local dialogueText = -1
local inDialogue = false
local dialogueLines = {}
local dialogueLine = 0

function onSceneCreationEnd()
    dialogueCanvas = UI.FindCanvas("Dialogue")
    if dialogueCanvas >= 0 then
        dialogueText = UI.FindElement(dialogueCanvas, "DialogueText")
        UI.SetCanvasVisible(dialogueCanvas, false)
    end
end

function startDialogue(lines)
    if inDialogue then return end
    inDialogue = true
    dialogueLines = lines
    dialogueLine = 1
    Controls.SetEnabled(false)
    UI.SetCanvasVisible(dialogueCanvas, true)
    UI.SetText(dialogueText, dialogueLines[1])
end

function advanceDialogue()
    if not inDialogue then return end
    dialogueLine = dialogueLine + 1
    if dialogueLine > #dialogueLines then
        endDialogue()
    else
        UI.SetText(dialogueText, dialogueLines[dialogueLine])
    end
end

function endDialogue()
    inDialogue = false
    Controls.SetEnabled(true)
    UI.SetCanvasVisible(dialogueCanvas, false)
end

function isInDialogue()
    return inDialogue
end

-- Publish all
_G.startDialogue = startDialogue
_G.advanceDialogue = advanceDialogue
_G.endDialogue = endDialogue
_G.isInDialogue = isInDialogue
```

### NPC Script

```lua
-- npc.lua
local talked = false

function onCreate(self)
    talked = false
end

function onInteract(self)
    if isInDialogue() then return end
    Interact.SetEnabled(self, false)

    if not talked then
        talked = true
        startDialogue({
            "Hello, traveler!",
            "Welcome to the world.",
            "Press CROSS to continue."
        })
    else
        startDialogue({
            "Hello again!",
            "Nothing new to say."
        })
    end
end

function onButtonPress(self, button)
    if not isInDialogue() then return end
    if button == Input.CROSS then
        advanceDialogue()
        if not isInDialogue() then
            Interact.SetEnabled(self, true)
        end
    end
end
```

---

## Pattern: Persistent Data Across Scenes

Use `Persist` to carry data between scene loads:

```lua
-- Before leaving the scene
Persist.Set("score", currentScore)
Persist.Set("health", currentHealth)
Persist.Set("came_from", Scene.GetIndex())
Scene.Load(1)

-- In the new scene
function onSceneCreationStart()
    local score = Persist.Get("score") or 0
    local health = Persist.Get("health") or 100
    local prevScene = Persist.Get("came_from") or -1
    Debug.Log("Arrived from scene " .. prevScene)
end
```

!!! warning "16 entry limit"
    Persist supports only 16 key-value pairs. Use it for important game state only.

---

## Pattern: Toggle Switch

Control another object by name lookup:

```lua
local target = nil
local isOn = false

function onCreate(self)
    target = Entity.Find("SwitchTarget")
    if target then
        Entity.SetActive(target, false)
    else
        Debug.Log("WARNING: SwitchTarget not found!")
    end
end

function onInteract(self)
    isOn = not isOn
    if target then
        Entity.SetActive(target, isOn)
    end
    Audio.Play(isOn and "switch_on" or "switch_off", 100, 64)
    setStatus(isOn and "Switch ON" or "Switch OFF")
end
```

---

## Pattern: Object Movement with Buttons

Move objects using button input with fixed-point step sizes:

```lua
local selected = false

function onInteract(self)
    selected = not selected
    if selected then
        Controls.SetEnabled(false)
        Interact.SetEnabled(self, false)
        setStatus("D-pad to move, L1/R1 for height, Triangle to deselect")
    else
        Controls.SetEnabled(true)
        Interact.SetEnabled(self, true)
    end
end

function onButtonPress(self, button)
    if not selected then return end

    local pos = self.position
    local step = FixedPoint.new(1) / 64

    if button == Input.UP then
        Entity.SetPosition(self, Vec3.new(pos.x, pos.y, pos.z + step))
    elseif button == Input.DOWN then
        Entity.SetPosition(self, Vec3.new(pos.x, pos.y, pos.z - step))
    elseif button == Input.LEFT then
        Entity.SetPosition(self, Vec3.new(pos.x - step, pos.y, pos.z))
    elseif button == Input.RIGHT then
        Entity.SetPosition(self, Vec3.new(pos.x + step, pos.y, pos.z))
    elseif button == Input.L1 then
        Entity.SetPosition(self, Vec3.new(pos.x, pos.y + step, pos.z))
    elseif button == Input.R1 then
        Entity.SetPosition(self, Vec3.new(pos.x, pos.y - step, pos.z))
    elseif button == Input.SQUARE then
        local rot = Entity.GetRotationY(self)
        local one = FixedPoint.new(1)
        Entity.SetRotationY(self, rot + one / 2)  -- Rotate 90 degrees
    elseif button == Input.TRIANGLE then
        selected = false
        Controls.SetEnabled(true)
        Interact.SetEnabled(self, true)
    end
end
```

---

## Pattern: Lua Free Cam

Disclaimer: onUpdate runs every frame so you can very easily lag out your project if you start using it for everything. 

Only use onUpdate if you really need to. This example script just demonstrates how you would make a smooth free cam but you could also modify it to have the camera jump to the next position/rotation in a large step with each button press and completely remove onUpdate.

```lua
-- Example Lua Free Cam --
-- Author: Latch
--
-- This script allows you to fully control the camera
-- with lua. You can also save the camera position+rotation
-- with square, move the camera somewhere else, and then
-- use circle to load the saved position+rotation
--
-- Warning!
-- PsxPlayer player will overwrite these values
--

local camRotStep = FixedPoint.new(1) / 128
local camMoveStep = FixedPoint.new(1) / 1024

local savedCamRotation = Vec3.new(0,0,0)
local savedCamPosition = Vec3.new(0,0,0)
 
function onUpdate(self, dt)

    local camPos = Camera.GetPosition()
    
    -- Camera Movement
    if Input.IsHeld(Input.UP) then
        Camera.MoveForward(camMoveStep)
    elseif Input.IsHeld(Input.DOWN) then
        Camera.MoveBackward(camMoveStep)
    elseif Input.IsHeld(Input.LEFT) then
        Camera.MoveLeft(camMoveStep)
    elseif Input.IsHeld(Input.RIGHT) then
        Camera.MoveRight(camMoveStep)
    elseif Input.IsHeld(Input.TRIANGLE) then
        Camera.SetPosition(Vec3.new(camPos.x,camPos.y-camMoveStep,camPos.z))
    elseif Input.IsHeld(Input.CROSS) then
        Camera.SetPosition(Vec3.new(camPos.x,camPos.y+camMoveStep,camPos.z))
        
    end

    local camRot = Camera.GetRotation()

    -- Camera Rotation
    if Input.IsHeld(Input.L1) then
        Camera.SetRotation(Vec3.new(camRot.x,camRot.y-camRotStep,camRot.z))   
    elseif Input.IsHeld(Input.R1) then
        Camera.SetRotation(Vec3.new(camRot.x,camRot.y+camRotStep,camRot.z))
    elseif Input.IsHeld(Input.L2) then
        Camera.SetRotation(Vec3.new(camRot.x-camRotStep,camRot.y,camRot.z)) 
    elseif Input.IsHeld(Input.R2) then
        Camera.SetRotation(Vec3.new(camRot.x+camRotStep,camRot.y,camRot.z))
    end
end

function onButtonPress(self, button)
    
    -- Camera Saving And Loading
    if button == Input.SQUARE then
        Debug.Log("Saving cam position and rotation")
        savedCamPosition = Camera.GetPosition()        
        savedCamRotation = Camera.GetRotation()
    elseif button == Input.CIRCLE then
        Debug.Log("Loading saved position and rotation")
        Camera.SetPosition(savedCamPosition);
        Camera.SetRotation(savedCamRotation);
    end
end

-- Debug.Log("Cam Position \n" .. "x" .. camPos.x .. " y" .. camPos.y .. " z" .. camPos.z)
```

---

## Pattern: PsxPlayer Position and Rotation

Control the PsxPlayer with lua functions

```lua
-- Example PsxPlayer Teleportation and Rotation --
-- Author: Latch
--
-- Multiple ways to teleport the player
-- Save Position+Rotation
-- Load Position+Rotation
-- Show Player Coordinates 
-- Player rotation by step amount

local savedPlayerPos = Vec3.new(0,0,0)
local savedPlayerRot = Vec3.new(0,0,0)

-- 1 is 180 Degrees
local rotStep = FixedPoint.new(1) / 4 
-- rotStep is 45 degrees

-- Location in world coordinates (int)
local locationX = -918; 
local locationY = 88;
local locationZ = 27;

function onButtonPress(self, button)
  
    -- Save Position and Rotation in a vec3
    if button == Input.SQUARE then 
        savedPlayerPos = Player.GetPosition()
        savedPlayerRot = Player.GetRotation()
        Debug.Log("\nSaved Player Position \n" .. "x:" .. savedPlayerPos.x .. " y:" .. savedPlayerPos.y .. " z:" .. savedPlayerPos.z)
        Debug.Log("Saved Player Rotation \n" .. "x:" .. savedPlayerRot.x .. " y:" .. savedPlayerRot.y .. " z:" .. savedPlayerRot.z .. "\n")

    -- Teleport to the player saved vec3 position+rotation 
    elseif button == Input.CIRCLE then  
        Player.SetPosition(savedPlayerPos)
        Player.SetRotation(savedPlayerRot)
        Debug.Log("Teleporting Player > Saved Vec3")

    -- Teleport to the location variable
    elseif button == Input.L1 then
        Player.SetPosition(locationX,locationY,locationZ)
        Debug.Log("Teleporting Player > Var Coords")
    
    -- Teleport to passed coordinates
    elseif button == Input.R1 then 
        Player.SetPosition(-93,88,329)
        Debug.Log("Teleporting Player > Given Coords")
    
    -- Show player coordinates 
    elseif button == Input.L2 then  
        local playerPos = Player.GetPosition()
        Debug.Log("\nPlayer Position FixedPoint\n" .. "x:" .. playerPos.x .. " y:" .. playerPos.y .. " z:" .. playerPos.z)
        Debug.Log("\nPlayer Position World / Int\n" .. "x:" .. playerPos.x*4096 .. " y:" .. playerPos.y*4096 .. " z:" .. playerPos.z*4096)

    -- Rotate the player 45 degrees to the right
    elseif button == Input.TRIANGLE then
        local playerRot = Player.GetRotation();
        Player.SetRotation(Vec3.new(playerRot.x,playerRot.y+rotStep,playerRot.z))
        Debug.Log("Rotating player 45 degrees")
    end
end
```

---

## Pattern: Cutscene Trigger Zone

One-time area trigger that plays a cutscene:

```lua
local played = false

function onTriggerEnter()
    if played then return end
    played = true
    Controls.SetEnabled(false)
    Cutscene.Play("camera_flyover", {
        onComplete = function()
            Controls.SetEnabled(true)
            setStatus("Cutscene complete!")
        end
    })
end
```

---

## Pattern: Health System

Color-coded health bar with damage and healing:

```lua
-- scene.lua
local health = 100
local healthBar = -1

function onSceneCreationEnd()
    local hud = UI.FindCanvas("HUD")
    healthBar = UI.FindElement(hud, "HealthBar")
    updateHealthBar()
end

function updateHealthBar()
    if healthBar < 0 then return end
    UI.SetProgress(healthBar, health)

    if health > 50 then
        UI.SetProgressColors(healthBar, 0, 40, 0, 0, 255, 0)        -- Green
    elseif health > 25 then
        UI.SetProgressColors(healthBar, 40, 40, 0, 255, 255, 0)     -- Yellow
    else
        UI.SetProgressColors(healthBar, 40, 0, 0, 255, 0, 0)        -- Red
    end
end

function applyDamage(amount)
    health = PSXMath.Clamp(health - amount, 0, 100)
    updateHealthBar()
    if health <= 0 then
        setStatus("You died! Reloading...")
        Controls.SetEnabled(false)
        Persist.Set("score", 0)
        Scene.Load(Scene.GetIndex())  -- Reload current scene
    end
end

function applyHeal(amount)
    health = PSXMath.Clamp(health + amount, 0, 100)
    updateHealthBar()
end

_G.applyDamage = applyDamage
_G.applyHeal = applyHeal
```

### Damage Trigger

```lua
function onTriggerEnter()
    applyDamage(25)
end
```

### Heal Trigger

```lua
function onTriggerEnter()
    applyHeal(50)
    Audio.Play("heal", 100, 64)
end
```

---

## Pattern: Scene Portal

Transition between scenes with data preservation:

```lua
-- portal_trigger.lua
function onTriggerEnter()
    setStatus("Loading next scene...")
    Persist.Set("came_from", Scene.GetIndex())
    Scene.Load(0)  -- Load scene 0
end
```

---

## Pattern: Looping Animation Toggle

Start/stop a looping animation on interact:

```lua
function onInteract(self)
    if Animation.IsPlaying("anim_spinner") then
        Animation.Stop("anim_spinner")
        setStatus("Spinner stopped!")
    else
        Animation.Play("anim_spinner", {loop = true})
        setStatus("Spinner started!")
    end
end
```

---

## Pattern: Entity Scanning / Diagnostics

Enumerate all entities for debugging:

```lua
function onInteract(self)
    Debug.Log("=== Entity Scan ===")
    Debug.Log("Total entities: " .. Entity.GetCount())

    Entity.ForEach(function(obj, index)
        local pos = Entity.GetPosition(obj)
        local active = Entity.IsActive(obj)
        Debug.Log("  [" .. index .. "] pos=" .. pos.x .. "," .. pos.y .. "," .. pos.z
            .. " active=" .. tostring(active))
    end)
end
```
