# Scene Components

Everything you can place in a SplashEdit scene. Each component is a Unity MonoBehaviour or ScriptableObject that tells SplashEdit what to export.

## Core Components

| Component | Purpose |
|-----------|---------|
| [Scene Exporter](scene-exporter.md) | Master controller for the scene export pipeline |
| [PSXObjectExporter](objects.md) | Makes a mesh renderable on PS1 |
| [PSXPlayer](player.md) | Player controller and camera |

## Visual

| Component | Purpose |
|-----------|---------|
| [Textures & VRAM](textures.md) | How textures are quantized and packed |
| [Custom Fonts](fonts.md) | Bitmap fonts for UI text |
| [Loading Screens](loading-screens.md) | Shown during scene transitions |

## Spatial

| Component | Purpose |
|-----------|---------|
| [Navigation & Collision](navigation.md) | Walkable surfaces and collision |
| [Rooms & Portals](rooms-portals.md) | Interior scene occlusion culling |

## Interactivity

| Component | Purpose |
|-----------|---------|
| [UI System](ui.md) | Canvases, text, images, progress bars |
| [Audio](audio.md) | Sound effects and music |
| [Interactables](interactables.md) | Button-press interactions |
| [Trigger Boxes](trigger-boxes.md) | Area-based event triggers |

## Sequencing

| Component | Purpose |
|-----------|---------|
| [Cutscenes](cutscenes.md) | Keyframed sequences with camera control |
| [Animations](animations.md) | Multi-instance object/UI animations |
