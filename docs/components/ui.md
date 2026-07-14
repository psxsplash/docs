# UI System

SplashEdit has a PS1-native UI system. You build UI using Unity's Canvas system, and SplashEdit exports the layout and assets into a custom binary format that the PS1 runtime renders natively.

## Creating a Canvas

1. Create a Canvas in your scene (**GameObject -> UI -> Canvas**)
2. Add a `PSXCanvas` component to it
3. SplashEdit auto-configures the canvas to match PS1 resolution (320x240)

### PSXCanvas Settings

| Field | Description | Default |
|-------|-------------|---------|
| Canvas Name | Unique name, max 24 characters. Used in Lua to find the canvas. | (required) |
| Start Visible | Whether the canvas is shown when the scene loads | true |
| Sort Order | Render order: 0 = back, 255 = front. Higher draws on top. | 0 |
| Default Font | Optional [custom font](fonts.md) for all text elements in this canvas | None |

## UI Element Types

Add these components to child GameObjects within a PSXCanvas. Position and size are controlled by the RectTransform.

### PSXUIImage

A textured image on the PS1 screen.

| Field | Description |
|-------|-------------|
| Element Name | Max 24 chars, used in Lua to find this element |
| Source Texture | The image to display |
| Bit Depth | 4-bit, 8-bit, or 16-bit |
| Tint Color | RGB tint applied to the image |
| Start Visible | Initial visibility |

!!! warning "Tint color brightness"
    Using a full white tint (255, 255, 255) will **overblow** PSX UI images, making them look washed out. Tone it down to around **128, 128, 128** for a normal appearance. Think of the tint as a multiplier, not a base color.

!!! danger "Image texture requirements"
    UI image textures must also be **power-of-two** in both dimensions, max **256x256**, just like object textures.

### PSXUIBox

A solid-color rectangle.

| Field | Description |
|-------|-------------|
| Element Name | Max 24 chars |
| Box Color | Fill color (RGB) |
| Start Visible | Initial visibility |

### PSXUIText

A text label rendered using the PS1 font system.

| Field | Description |
|-------|-------------|
| Element Name | Max 24 chars |
| Default Text | Initial text, max 63 characters |
| Text Color | RGB color |
| Font Override | Use a specific [custom font](fonts.md) instead of the canvas default |
| Start Visible | Initial visibility |

Text supports ASCII characters 0x20 through 0x7F (standard printable characters: space through tilde).

### PSXUIProgressBar

A two-part bar with background and fill colors.

| Field | Description |
|-------|-------------|
| Element Name | Max 24 chars |
| Background Color | Color of the empty portion |
| Fill Color | Color of the filled portion |
| Initial Value | Starting fill percentage, 0-100 |
| Start Visible | Initial visibility |

### PSXUILine

A straight colored line between two points (rendered as a GPU line primitive). Add it via **Add Component -> PSX/UI/PSX UI Line**.

| Field | Description |
|-------|-------------|
| Element Name | Max 24 chars, used in Lua to find this element |
| Line Color | Line color (RGB) |
| Point 1 | First endpoint, in PS1 pixel coordinates |
| Point 2 | Second endpoint, in PS1 pixel coordinates |
| Start Visible | Initial visibility |

Unlike the other elements, a line is defined by its two **endpoints** rather than a position + size, so its layout comes from the `Point 1` / `Point 2` fields. You can still toggle its visibility and change its color from Lua like any other element (`UI.SetVisible`, `UI.SetColor`). `UI.GetElementType` returns `4` for a line.

!!! tip "Lines vs. immediate-mode drawing"
    Use `PSXUILine` for lines that are part of an authored layout (HUD frames, dividers, gauges). For lines you compute on the fly each frame — debug rays, aim indicators, markers over 3D objects — use [`UI.DrawLine`](#drawing-directly-from-lua) instead.

## Coordinate System

UI elements use **PS1 pixel coordinates** (320x240 resolution). Position and size come from the RectTransform in Unity. SplashEdit converts the Unity layout to PS1 coordinates at export time.

## Controlling UI from Lua

```lua
-- Find a canvas and elements
local hud = UI.FindCanvas("HUD")
local scoreText = UI.FindElement(hud, "ScoreText")
local healthBar = UI.FindElement(hud, "HealthBar")

-- Show/hide canvases
UI.SetCanvasVisible(hud, true)
UI.SetCanvasVisible("Dialogue", false)  -- accepts name or index

-- Update text (max 63 characters)
UI.SetText(scoreText, "Score: 500")

-- Update progress bar (0-100)
UI.SetProgress(healthBar, 75)

-- Change element color (RGB 0-255)
UI.SetColor(scoreText, 255, 255, 0)  -- yellow

-- Change progress bar colors (background RGB, fill RGB)
UI.SetProgressColors(healthBar, 0, 40, 0, 0, 255, 0)

-- Move/resize elements
UI.SetPosition(element, 10, 20)
UI.SetSize(element, 100, 32)

-- Check visibility
if UI.IsCanvasVisible(hud) then
    -- ...
end
```

See the [complete API reference](../lua/api-reference.md#ui) for all UI functions.

## Drawing Directly from Lua

Besides the authored elements above, Lua can draw **immediate-mode** primitives straight to the screen each frame — no Canvas element required. This is ideal for things you generate dynamically: debug overlays, aim lines, or markers pinned to moving 3D objects.

```lua
function onUpdate(self, dt)
    -- A line and a gouraud-shaded triangle, in PS1 pixel coords (320x240)
    UI.DrawLine({10, 120}, {310, 120}, {255, 0, 0})
    UI.DrawTriangle(
        {160, 40}, {120, 120}, {200, 120},
        {255, 0, 0}, {0, 255, 0}, {0, 0, 255})
end
```

These primitives are **not** persistent — they're drawn for the current frame only, so you must re-issue them every frame (typically from `onUpdate`). Combine with [`PSXMath.Convert3DTo2D`](../lua/api-reference.md#psxmath) to anchor them to a world position. See the [API reference](../lua/api-reference.md#immediate-mode-drawing) for exact signatures.

## Example: HUD with Score and Health

```
Canvas "HUD" (startVisible: true, sortOrder: 10)
  |-- PSXUIText "ScoreText" (default: "Score: 0")
  |-- PSXUIText "StatusText" (default: "")
  |-- PSXUIProgressBar "HealthBar" (initial: 100)
```

```lua
-- In scene.lua
function onSceneCreationEnd()
    local hud = UI.FindCanvas("HUD")
    local scoreText = UI.FindElement(hud, "ScoreText")
    local healthBar = UI.FindElement(hud, "HealthBar")

    UI.SetCanvasVisible(hud, true)
    UI.SetText(scoreText, "Score: 0")
    UI.SetProgress(healthBar, 100)
end
```
