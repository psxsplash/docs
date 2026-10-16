# Tutorial: Collectible Pickups

Objects the player walks into to collect, with a score on the HUD and a sound.

You will use `onCollideWithPlayer`, `Entity.SetActive`, `Audio.Play`, `Persist` and UI text.

## 1. The HUD

1. Open the **UI** tab. In the canvas picker choose **New canvas** and name it `hud` in the inspector.
2. Click **+** in the element list and add a **Text**. Name it `score` and set its text to `Score: 0`. Drag it to the top left.

## 2. The scene script

Select nothing, and under **Script** in the scene settings click **New Lua**. Paste:

```lua
local scoreText = -1

function onSceneCreationEnd()
    local hud = UI.FindCanvas("hud")
    if hud >= 0 then
        scoreText = UI.FindElement(hud, "score")
        updateScoreDisplay()
    end
end

function updateScoreDisplay()
    if scoreText >= 0 then
        UI.SetText(scoreText, "Score: " .. (Persist.Get("score") or 0))
    end
end

function addScore(amount)
    Persist.Set("score", (Persist.Get("score") or 0) + amount)
    updateScoreDisplay()
end

-- Publish so object scripts can call these
_G.addScore = addScore
```

## 3. The pickup

1. Add a **Mesh** object (a small crate or gem) and give it a **Collider** with shape **Dynamic**. Only dynamic colliders are tested against the player, so only they fire `onCollideWithPlayer`.
2. On the object click **New Lua** in its Script component, and paste:

```lua
local collected = false

function onCollideWithPlayer(self)
    if collected then return end
    collected = true

    Audio.Play("collect", 127, 64)
    Entity.SetActive(self, false)
    addScore(100)
end
```

3. Duplicate the object (**Ctrl+D**) a few times and place the copies around the scene. Each copy has its own `collected` flag.

## 4. The sound

Add an **Audio clip** object, pick a short WAV, and set **Name** to `collect`. See [Audio](../editor/audio.md).

## 5. Play

Press **Play** (**F5**), click the game, and walk into a pickup. It disappears, plays the sound, and the score goes up by 100.

## How it works

1. `onCollideWithPlayer` fires every frame the player overlaps the object. The `collected` flag makes it run once.
2. `Entity.SetActive(self, false)` hides the object.
3. `addScore` runs in the scene script, because it was published to `_G`. It stores the score in `Persist`, which survives scene loads, and updates the text.

## With blocks

The pickup is a **when the player touches me** stack: `if not collected` then `set collected to true`, `play sound collect`, `hide self`, `change score by 100`. Make `collected` and `score` variables of that script. The score would then live in that script instead of `Persist`, so use Lua for the scene script if other scenes need the score.
