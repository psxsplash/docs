# Quick Start: Your First PS1 Scene

This walkthrough creates a minimal scene with a floor, an object, and a player. By the end you'll have something running in the emulator.

## 1. Create a New Project

Create a new Unity project using the **Universal 3D** template. This gives you a Universal Render Pipeline project, which SplashEdit requires.

## 2. Install SplashEdit

Follow the [Installation](installation.md) guide to install the SplashEdit package and toolchain.

## 3. Create a Scene

Create a new Unity scene (**File -> New Scene**).

## 4. Add a Scene Exporter

Go to **GameObject -> PlayStation 1 -> Scene Exporter**. This adds a GameObject with the `PSXSceneExporter` component. There can only be one per scene - it's the master controller for the export pipeline.

## 5. Add a Floor

!!! warning "Don't use Unity's built-in Plane"
    Unity's default Plane is two huge triangles. On the PS1, large polygons cause severe texture warping and will likely get culled entirely. Instead, **create your floor geometry in a 3D modeling tool** (Blender, etc.) and subdivide it into small quads/triangles. Built-in Unity primitives like Cube can work for simple testing, but avoid Plane, Sphere, Cylinder, and other complex built-in meshes.

1. Import a subdivided floor mesh from your 3D tool
2. Assign a material with a texture
3. Add the `PSXObjectExporter` component to it
4. In the component set Collision>Type to Static

!!! danger "Texture Requirements"
    All textures **must be power-of-two in both width and height**, and the maximum size is **256x256**. Valid sizes: 16x16, 32x32, 64x64, 128x128, 256x256 (and non-square like 64x128). SplashEdit does NOT check this for you - incorrect textures will produce broken results.

## 6. Add a Player

1. Create an empty GameObject
2. Add the `PSXPlayer` component
3. Position it above the floor

The player is invisible at runtime - it represents the camera and controller.

## 7. Add the Nav Region

1. Open the PlayStation 1 > Nav Region Builder
2. Click **Build Nav Regions**

Note: Closing the Nav Region Builder window will hide the visual but it's still there

## 8. Add the Scene to the Build

1. Open the Control Panel (++ctrl+shift+l++)
2. Go to the **Scenes** tab
3. Click **"+ Add Current Scene"**

## 9. Build and Run

1. Go to the **Build** tab
2. Set Target to **Emulator**
3. Click **BUILD & RUN**

SplashEdit exports the scene, compiles the PS1 executable, and launches PCSX-Redux. You should see your textured floor with camera controls.

## 10. Enjoy

You should now be able to walk around. Left joystick for movement and right joystick for camera controls. If you aren't getting proper input. See the trouble shooting section below.

## What Just Happened?

Behind the scenes, SplashEdit:

1. Quantized your texture to PS1-compatible color depth
2. Packed it into VRAM alongside the framebuffers and system font
3. Converted your mesh to fixed-point PS1 vertex format with pre-baked lighting
4. Wrote everything into a `.splashpack` binary
5. Compiled the psxsplash C++ runtime with your configuration
6. Launched PCSX-Redux serving your data via PCdrv

## Next Steps

- Add more objects with different textures and bit depths
- [Set up UI](../components/ui.md) with a HUD canvas
- [Write Lua scripts](../lua/index.md) for game logic
- [Add interactables](../components/interactables.md) the player can interact with
- Check out the [tutorials](../tutorials/index.md) for complete game mechanics


## Trouble Shooting

Problem: 
The game runs but I can't move around with the Joysticks. Inputs don't work / are wrong

Solution 1:

When your game launches there should be some tabs at the top. If you don't see them press ESC to bring them up. Then go to
Configuration > Controls
Make sure "Controller Type" is analog and "Analog Mode" is checked. Also if you are using an original controller you may have to physically click the "ANALOG" button between the joysticks to get proper input.

Solution 2: 

If that still doesn't work review steps 5, 6, and 7 for a step that was missed.