# Scene Settings

Select nothing (**Edit > Select None**, or click empty space in the viewport) and the inspector shows the scene's settings, one section after another.

## Scene script

**File** is a script, blocks or Lua, that runs for the whole scene, with **New blocks**, **New Lua** and **Edit script** like any Script component. A Lua scene script gets `onSceneCreationStart` and `onSceneCreationEnd`; a block one starts with **when the scene starts**. See [Event Callbacks](../lua/events.md#scene-callbacks).

## Cutscenes and animations

Every cutscene and animation in the scene, each with **Edit in timeline**. See [Cutscenes & Animations](timeline.md).

## Rendering

| Field | Meaning |
|-------|---------|
| **Background** | The colour the screen clears to every frame. With fog or Edge fade on, distant triangles fade to it. |
| **Fog** | Fades distant triangles to the background colour, and culls anything wholly past the fog wall. |
| **Density** | Fog density, 1 to 10. Kept while fog is off. |
| **Edge fade** | On by default. Distant objects fade in from the background colour as they come within drawing distance, instead of appearing all at once. Works with fog on or off. |
| **GTE scaling** | Editor units per PS1 unit, default 100. Positions export as value divided by this, in 16-bit fixed point, so everything must sit within 8 x the scaling of the origin: 800 m at 100. Smaller is finer, larger reaches farther. |
| **Scene type** | Exterior or Interior. Stored in the scene, but the engine does not read it: culling follows [Room objects](navigation-rooms.md#rooms-and-portals). |

A caption underneath says which culling the scene will use.

## Loading screen

**Canvas** picks a [UI canvas](ui-canvas.md) of this scene that psxsplash draws while the scene loads. The canvas is left out of the scene itself.

!!! note "The progress bar must be named `loading`"
    The engine fills only a progress bar whose **Name** is exactly `loading`, from 0 to 100 as the scene loads. A bar with any other name stays at its initial value, and the settings warn you. The loading screen is drawn directly while the scene loads, so animation tracks and `UI.SetProgress` do not run on it. **Line** elements are not drawn on it either.

## Build scenes

The scenes that Play and Export write, in order. **Add scene** picks from the `.scene` files in the project; the arrows reorder and the **x** takes one out. Play and Export start at scene 0, and `Scene.Load(i)` opens scene *i* of this list. With an empty list, the open scene is written alone.

## VRAM layout

A map of the 1024 x 512 VRAM: the two framebuffers, the font column (x 960) and the texture atlases. **Prohibited areas** are rectangles you drag out for textures and palettes to stay out of, for example to keep room for something your game draws itself. Without any, the packer may use all of VRAM except the framebuffers and x 960.

## Disc

**Volume label** and **Licence** for exported disc images. See [Play, Hardware and Discs](../getting-started/play-and-export.md#export-a-disc-image).

## Render buffers

Two sizes the console allocates for drawing. Both start at **auto**, sized from the scene; type a number to override, and the undo arrow goes back to auto.

| Buffer | Meaning |
|--------|---------|
| **Ordering table** | Depth slots the renderer sorts triangles into, back to front. One slot per depth step, so this is also how far the camera sees: anything wholly beyond it is not drawn. 8 bytes of RAM per slot. |
| **Primitive buffer** | Bytes of GPU commands one frame can build: every triangle drawn, plus sprites and text. If a frame needs more, the game stops with "Out of memory". It costs twice its size in RAM, once per frame being drawn. |

While the game runs under Play, the settings add the peaks it actually reached: the farthest depth drawn and the busiest frame's bytes. The section warns when the scene may cut off or run out.

## BVH

The bounding volume hierarchy psxsplash culls with: it draws only the triangles in boxes the camera can see. The section reports the node, leaf and triangle-reference counts. **Show in viewport** draws the boxes over the scene, one colour per level, and **Depth** sets the deepest level drawn (0 is the root box around the whole scene).

## Navigation

The nav bake's region and portal counts, with the viewport overlays. See [Player, Navigation & Rooms](navigation-rooms.md#seeing-the-bake).

## World streaming

See [World Streaming](world-streaming.md).

## Memory card

For `MemCard.Save` and friends. **Enable saves** packs the settings into this scene; off, the console keeps the last scene's settings, so give every scene of a game the same values.

| Field | Meaning |
|-------|---------|
| **Region** | `BA` America, `BE` Europe, `BI` Japan. Starts every save filename. |
| **Product code** | For example `SLUS-00000`, up to 10 characters. |
| **Save title** | What the BIOS memory card screen shows, ASCII only, at most 32 characters. `"`, `'`, `-`, `\` and `~` show as `?`. `MemCard.Save`'s fourth argument overrides it per save. |
| **Icon** | One to three 16 x 16 images. Two or three animate. |

A save's filename is region + product code + the key you pass to `MemCard.Save`, cut at 20 characters. With a 10-character product code only the first 8 characters of a key count; the settings warn when region and product fill all 20 and every key would save to the same file. A save made under one product code is not found under another.
