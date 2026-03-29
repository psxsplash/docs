# Tutorial: Collectible Pickups

Build objects the player walks into to collect, with score tracking and sound effects.

## What You'll Learn

- `onCollideWithPlayer` callback
- `Entity.SetActive` to hide objects
- `Audio.Play` for sound effects
- `Persist` for score tracking
- UI text updates
- Scene-to-object function sharing via `_G`

## Step 1: Create the HUD

1. Create a Canvas (**GameObject -> UI -> Canvas**)
2. Add `PSXCanvas` with name `"HUD"`, startVisible = true
3. Create a child Text element, add `PSXUIText` with name `"ScoreText"` and default text `"Score: 0"`

## Step 2: Scene Script

Create `scene.lua` and assign it to the Scene Exporter's **Scene Lua File**:

```lua
local hudCanvas = -1
local scoreText = -1

function onSceneCreationEnd()
    hudCanvas = UI.FindCanvas("HUD")
    if hudCanvas >= 0 then
        scoreText = UI.FindElement(hudCanvas, "ScoreText")
        UI.SetCanvasVisible(hudCanvas, true)
        updateScoreDisplay()
    end
end

function updateScoreDisplay()
    if scoreText >= 0 then
        local score = Persist.Get("score") or 0
        UI.SetText(scoreText, "Score: " .. score)
    end
end

function addScore(amount)
    local score = (Persist.Get("score") or 0) + amount
    Persist.Set("score", score)
    updateScoreDisplay()
end

function setStatus(msg)
    Debug.Log("[Status] " .. msg)
end

-- Publish so object scripts can call these
_G.addScore = addScore
_G.setStatus = setStatus
_G.updateScoreDisplay = updateScoreDisplay
```

## Step 3: Collectible Script

Create `collectible.lua`:

```lua
local collected = false

function onCreate(self)
    collected = false
    Debug.Log("Collectible created at "
        .. self.position.x .. "," .. self.position.y .. "," .. self.position.z)
end

function onCollideWithPlayer(self)
    if collected then return end
    collected = true

    Audio.Play("collect", 127, 64)
    Entity.SetActive(self, false)

    addScore(100)
    setStatus("Collected! +100 points")
end
```

## Step 4: Create Collectible Objects

1. Create 3 small mesh GameObjects (cubes, spheres, gems, etc.)
2. Add `PSXObjectExporter` to each
3. Assign `collectible.lua` to each one's **Lua File** field
4. Position them around the scene where the player can walk into them

!!! tip
    Each object gets its own instance of the script with its own `collected` flag. You can place as many collectibles as you want using the same script.

## Step 5: Add Audio

1. Add a `PSXAudioClip` component to any GameObject in the scene
2. Set **Clip Name** to `"collect"`
3. Assign a short pickup sound to the **Clip** field

## Step 6: Build and Test

1. Open the Control Panel (++ctrl+shift+l++)
2. Click **BUILD & RUN**
3. Walk into each collectible - you should hear the sound, see the object disappear, and see the score update

## How It Works

1. When the player's collision capsule overlaps a collectible, `onCollideWithPlayer` fires
2. The `collected` flag prevents the callback from running more than once
3. `Audio.Play("collect", 127, 64)` plays the sound at full volume, centered
4. `Entity.SetActive(self, false)` hides the object (triggers `onDisable`)
5. `addScore(100)` is a scene-level function that updates Persist storage and the UI text

## Next Steps

- Add different point values for different collectible types
- Display a status message when all collectibles are found
- Add visual feedback (UI text flash, progress bar update)
