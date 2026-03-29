# Tutorial: Health & Damage

Create hazard zones that damage the player, healing zones that restore health, and a color-coded health bar.

## What You'll Learn

- Progress bar UI for health display
- `PSXMath.Clamp` for bounded values
- `UI.SetProgress` and `UI.SetProgressColors`
- Trigger boxes for damage/heal zones
- Death and scene reload

## Step 1: Health Bar UI

1. In your HUD canvas, create a child element
2. Add `PSXUIProgressBar` with:
    - Element Name: `"HealthBar"`
    - Initial Value: 100
    - Background Color: dark green (0, 40, 0)
    - Fill Color: bright green (0, 255, 0)

## Step 2: Health System in Scene Script

Add to your `scene.lua`:

```lua
local health = 100
local healthBar = -1

-- In onSceneCreationEnd, add:
-- healthBar = UI.FindElement(hudCanvas, "HealthBar")
-- UI.SetProgress(healthBar, 100)
-- updateHealthBar()

function updateHealthBar()
    if healthBar < 0 then return end
    UI.SetProgress(healthBar, health)

    -- Color-code based on health level
    if health > 50 then
        -- Green
        UI.SetProgressColors(healthBar, 0, 40, 0, 0, 255, 0)
    elseif health > 25 then
        -- Yellow (warning)
        UI.SetProgressColors(healthBar, 40, 40, 0, 255, 255, 0)
    else
        -- Red (danger)
        UI.SetProgressColors(healthBar, 40, 0, 0, 255, 0, 0)
    end
end

function applyDamage(amount)
    health = PSXMath.Clamp(health - amount, 0, 100)
    updateHealthBar()

    if health <= 0 then
        setStatus("You died! Reloading...")
        Controls.SetEnabled(false)
        Persist.Set("score", 0)       -- Reset score on death
        Scene.Load(Scene.GetIndex())  -- Reload current scene
    end
end

function applyHeal(amount)
    health = PSXMath.Clamp(health + amount, 0, 100)
    updateHealthBar()
end

-- Publish
_G.applyDamage = applyDamage
_G.applyHeal = applyHeal
_G.updateHealthBar = updateHealthBar
```

## Step 3: Damage Zone

1. Create an empty GameObject in a hazard area (lava, spikes, etc.)
2. Add `PSXTriggerBox`, set size to cover the area

Create `damage_trigger.lua`:

```lua
function onTriggerEnter()
    applyDamage(25)
    Debug.Log("Player took 25 damage!")
end
```

## Step 4: Healing Zone

1. Create an empty GameObject in a safe area
2. Add `PSXTriggerBox`, set size

Create `heal_trigger.lua`:

```lua
function onTriggerEnter()
    applyHeal(50)
    Audio.Play("heal", 100, 64)
    Debug.Log("Player healed 50 HP!")
end
```

## Step 5: Build and Test

Walk into the damage zone to see health decrease and the bar change color. Walk into the heal zone to restore. If health hits 0, the scene reloads.

## How It Works

1. Trigger boxes fire `onTriggerEnter` when the player enters
2. `applyDamage` / `applyHeal` use `PSXMath.Clamp` to keep health in 0-100 range
3. `updateHealthBar` sets both the progress value and the colors based on thresholds
4. At 0 health, `Scene.Load(Scene.GetIndex())` reloads the current scene (a simple death/restart)

## Customization Ideas

- **Invincibility frames**: Track a cooldown flag to prevent rapid re-damage
- **Health pickups**: Use `onCollideWithPlayer` on objects instead of trigger zones
- **Different damage amounts**: Create different trigger scripts with different values
- **Visual feedback**: Flash the health bar color or show a status message on hit
