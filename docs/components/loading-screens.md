# Loading Screens

Loading screens are shown while a scene loads. They're exported as a separate lightweight binary that loads instantly before the main scene data.

## Setup

1. Create a **Prefab** containing a `PSXCanvas` with your loading screen UI
2. On the [Scene Exporter](scene-exporter.md), assign this prefab to the **Loading Screen Prefab** field
3. The loading screen can contain any UI elements: images, text, progress bars, boxes

## How It Works

When a scene loads:

1. The lightweight `.loading` file is read first (fast)
2. The loading screen UI is displayed
3. The main `.splashpack` file loads in the background
4. Once loading completes, the loading screen disappears and gameplay begins

## Creating a Loading Screen Prefab

1. Create a new empty GameObject
2. Add a `PSXCanvas` component with your desired layout
3. Add child elements:
    - `PSXUIImage` for a background or logo
    - `PSXUIText` for "Loading..." text
    - `PSXUIProgressBar` for a loading indicator
    - `PSXUIBox` for solid-color backgrounds
4. Save as a Prefab

The loading screen uses the same [UI system](ui.md) and [font pipeline](fonts.md) as regular canvases.

## Sharing Loading Screens

Multiple scenes can reference the same loading screen prefab. The exporter detects identical loading screens and deduplicates the binary output.

## Tips

!!! tip "Keep it simple"
    Loading screens need to load instantly, so keep them lightweight. A solid background with text works well. Avoid large textures.

!!! tip "Loading screens have separate VRAM"
    The loading screen's textures and fonts are loaded into VRAM independently from the main scene. They don't count against your scene's VRAM budget.
