# Tutorial: Health & Damage

Hazard and healing zones and a health bar that changes colour.

You will use a UI **Progress bar**, `UI.SetProgress`, `UI.SetProgressColors`, `PSXMath.Clamp` and Trigger boxes.

## 1. The health bar

In the **UI** tab, make a canvas named `hud` if you do not have one, **+** a **Progress bar**, and name it `health`. Set its **Value** to 100 and place it near a corner.

## 2. Health in the scene script

Select nothing and **New Lua** under **Script**. Paste:

```lua
local health = 100
local healthBar = -1

function updateHealthBar()
    if healthBar < 0 then return end
    UI.SetProgress(healthBar, health)

    if health > 50 then
        UI.SetProgressColors(healthBar, 0, 40, 0, 0, 255, 0)      -- green
    elseif health > 25 then
        UI.SetProgressColors(healthBar, 40, 40, 0, 255, 255, 0)   -- yellow
    else
        UI.SetProgressColors(healthBar, 40, 0, 0, 255, 0, 0)      -- red
    end
end

function onSceneCreationEnd()
    local hud = UI.FindCanvas("hud")
    if hud >= 0 then
        healthBar = UI.FindElement(hud, "health")
        updateHealthBar()
    end
end

function applyDamage(amount)
    health = PSXMath.Clamp(health - amount, 0, 100)
    updateHealthBar()
    if health <= 0 then
        Controls.SetEnabledPlayer1(false)
        Scene.Load(Scene.GetIndex())    -- reload this scene
    end
end

function applyHeal(amount)
    health = PSXMath.Clamp(health + amount, 0, 100)
    updateHealthBar()
end

_G.applyDamage = applyDamage
_G.applyHeal = applyHeal
```

## 3. A damage zone

**Add Object > Trigger box**, size it over the hazard, and **New Lua** on its Script field:

```lua
function onTriggerEnter()
    applyDamage(25)
end
```

## 4. A healing zone

Another Trigger box in a safe spot:

```lua
function onTriggerEnter()
    applyHeal(50)
end
```

## 5. Play

Walk into the damage zone: the bar drops and goes yellow, then red. Walk into the healing zone to refill it. At zero the scene reloads.

## How it works

1. `onTriggerEnter` fires when the player walks into the box.
2. `PSXMath.Clamp` keeps health between 0 and 100.
3. `updateHealthBar` sets the fill and the colours from thresholds.
4. At zero, `Scene.Load(Scene.GetIndex())` reloads the current scene, a simple restart. The scene script runs again, so `health` is back to 100.

## Ideas

- A cooldown flag stops a zone from hurting every time the player steps in.
- Use `onCollideWithPlayer` on a Dynamic collider for a pickup that heals.
- Show a message on a Text element when health runs out.
