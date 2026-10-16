# Memory & Performance

The PS1 has little of everything, and psxsplash games run into the same walls. The editor shows where you stand against each wall while you work.

## Hardware limits

| Resource | Limit |
|----------|-------|
| Main RAM | 2 MB, shared by engine, scene data, render buffers and the Lua VM |
| [VRAM](glossary.md#vram) | 1 MB (1024 x 512 at 16 bits) |
| [SPU](glossary.md#spu-sound-processing-unit) RAM | 512 KB of audio |
| CPU | 33 MHz MIPS R3000A, no FPU |
| SPU voices | 24 at once |

## The status bar meters

At the bottom of the editor, for the scene as it would export right now:

| Meter | Counts |
|-------|--------|
| **VRAM** | Out of 1024 KB: two 320 x 240 framebuffers, the font column, texture atlases and their palettes ([CLUTs](glossary.md#clut-color-look-up-table)) |
| **SPU RAM** | Out of 512 KB: your [audio clips](../editor/audio.md) |
| **RAM** | Main RAM left for the scene after the engine: scene data, render buffers, streaming slots, and a per-object cost |
| **Triangles** | Triangles in the scene (not the number visible at once) |

The numbers come from the same export Play uses, so they follow every edit. The [scene settings](../editor/scene-settings.md#vram-layout) show where textures sit in VRAM.

## Where main RAM goes

- **Scene data**: objects, meshes, collision, navigation, UI, cutscenes, animations, skinned clips, Lua bytecode.
- **Render buffers**: two [ordering tables](glossary.md#ot-ordering-table) and two [primitive buffers](glossary.md#bump-allocator), sized from the scene. See [Render buffers](../editor/scene-settings.md#render-buffers).
- **Per object**: every object costs RAM at load whether or not it has a script: a few hundred bytes for its runtime state, plus two Lua tables. A scene of a thousand-plus objects can run out of RAM on that alone. The editor warns **Too many objects for RAM** and can select the biggest group. Merging objects (the **Merge meshes** import setting) fixes it.
- **Streaming slots**, when [world streaming](../editor/world-streaming.md) is on.
- **Lua VM and engine state.**

### Fixed pools

The engine reserves fixed-size pools for these, which is also why the editor enforces the limits:

| Pool | Size |
|------|------|
| UI canvases | 24 |
| UI elements | 256 across the scene |
| Cutscenes | 16 per scene, 8 tracks each |
| Animations | 8 playing at once |
| Rooms | 31 per scene |
| Runtime lights | 16 per scene, 4 per mesh |
| `Persist` entries | 16 numbers |

## Keeping to budget

### Textures (VRAM)

1. **Use 4 bpp when the texture allows.** The inspector offers it when a texture has 16 colours or fewer. See [Materials & Bit Depth](../editor/materials.md).
2. **Keep textures small.** 64 x 64 and 128 x 128 are normal PS1 sizes; 256 x 256 is the maximum.
3. **Reuse textures.** Each distinct texture takes VRAM.

### Geometry

1. **Use rooms and portals** for indoor scenes, so only visible rooms are drawn. See [Rooms and portals](../editor/navigation-rooms.md#rooms-and-portals).
2. **Use fog** to shorten how far the console draws.
3. **Watch the primitive buffer.** If a frame needs more than it holds, the game stops with "Out of memory". The settings show the busiest frame under Play.
4. **Check FPS** with **Build > Performance Overlay**.

### Skinned meshes (RAM)

Each frame of each clip stores 24 bytes per bone. Fewer bones, shorter clips and a lower **Bake rate** all shrink it. The inspector's **Baked** block lists the size of every clip.

### Audio (SPU RAM)

ADPCM takes about 0.57 bytes per sample, so a 5-second clip at 22050 Hz is about 63 KB. Lower the **Sample rate**, keep clips short, and plan for 24 voices at most.

### Lua (RAM and CPU)

1. **React to events instead of polling.** `onUpdate` runs every frame for every object that defines it.
2. **Define only the callbacks you use.** The engine checks a bitmask; undefined callbacks cost nothing.
3. **`Persist` holds 16 numbers.** For anything larger or longer-lived, use the memory card API.
