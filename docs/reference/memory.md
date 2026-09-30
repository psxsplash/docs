# Memory & Performance

## PS1 Hardware Limits

| Resource | Limit |
|----------|-------|
| Main RAM | 2MB total (engine + scene data + Lua VM) |
| [VRAM](glossary.md#vram) | 1MB (1024x512 at 16-bit) |
| [SPU](glossary.md#spu-sound-processing-unit) RAM | 512KB (audio samples) |
| CPU | 33MHz MIPS R3000A, no FPU, no cache |
| GPU Triangles | ~1000-2000 visible per frame at 30fps |
| SPU Voices | 24 simultaneous audio channels |

## Memory Report

After exporting from the [Control Panel](../getting-started/control-panel.md), a **Memory Report** foldout shows per-scene breakdowns:

### Main RAM Breakdown

| Section | Description |
|---------|-------------|
| [Ordering Table (OT)](glossary.md#ot-ordering-table) | Depth sorting buckets - configurable via OT Size |
| [Bump Allocator](glossary.md#bump-allocator) | Per-frame GPU command buffer - configurable via Bump Alloc Size |
| Scene Data | Objects, nav regions, collision, UI, cutscenes, animations, skinned mesh data |
| Other | Lua VM, engine state, stack |

### VRAM Breakdown

| Section | Description |
|---------|-------------|
| Framebuffers | Two 320x240 buffers (fixed) |
| Texture Atlases | Your scene textures |
| [CLUTs](glossary.md#clut-color-look-up-table) | Color palettes for 4-bit and 8-bit textures |
| Font Column | System font + up to 3 custom fonts |

### Fixed-Size Pools

A few subsystems reserve a fixed-size pool at compile time instead of
allocating per-scene. These are engine constants, not configurable from Unity.
Per-entry sizes below were computed by compiling each struct's field layout
standalone with the PS1's 32-bit pointers; the real MIPS compiler may pad
slightly differently, so treat anything marked **approx.** as an estimate
rather than an exact figure.

| Pool | Constant | Count | Per entry | Total |
|------|----------|-------|-----------|-------|
| Sprites | `SPRITE_MAX` (`spritesystem.hh`) | 128 | approx. 48 bytes (`SpriteInstance`) | approx. 6 KB |
| UI elements | `UI_MAX_ELEMENTS` (`uisystem.hh`) | 256 | approx. 108 bytes (`UIElement`, includes a per-type union) | approx. 27 KB - the engine's own comment in `uisystem.hh` estimates ~29 KB for this pool |
| UI canvases | `UI_MAX_CANVASES` (`uisystem.hh`) | 24 | approx. 16 bytes (`UICanvas`) | approx. 384 bytes |
| [SIO1](glossary.md#sio1) RX ring | `Sio1::c_rxRingSize` (`sio1.hh`) | 4096 bytes of payload | + 8 bytes of head/tail indices, exactly | 4104 bytes |
| SIO1 TX ring | `Sio1::c_txRingSize` (`sio1.hh`) | 1024 bytes of payload | + 8 bytes of head/tail indices, exactly | 1032 bytes |
| Per-agent runtime state | `AgentRuntimeState` (`scenemanager.hh`) | one per agent actor - an `eastl::vector` sized at scene load, no fixed max | approx. 224 bytes | approx. 224 bytes x agent count |

!!! note "Sizes at time of writing"
    These numbers reflect the engine source as of this doc pass and will drift
    as the structs change. If you need an exact figure, check the struct
    definitions directly rather than trusting this table long-term.

### Warning Thresholds

!!! danger "Memory report is very approximate"
    The memory preview in the Control Panel is **very limited and very approximate**. Do not trust the displayed percentages as accurate. A safe rule of thumb: if it shows **50% usage**, you're probably fine. Anything higher and you should test carefully on real hardware or emulator.

- **Yellow** at 80% displayed - you're likely already at the limit
- **Red** at 90% displayed - almost certainly over budget

## Optimization Tips

### Textures (VRAM)

1. **Use 4-bit aggressively.** 16 colors with dithering looks good on PS1. Saves 4x VRAM vs 16-bit.
2. **Share materials.** Identical textures are deduplicated, but different materials pointing to the same image are not.
3. **Keep textures small.** 64x64 and 128x128 are standard PS1 sizes.
4. **Reduce unique textures.** Each unique texture occupies VRAM. Reuse materials across objects.

### Geometry (RAM + GPU)

1. **Target 1000-2000 visible triangles per frame.** The GPU can handle more in theory, but the CPU overhead of transforming vertices is the bottleneck.
2. **Use Interior scenes with rooms/portals.** Only visible rooms are rendered, dramatically reducing triangle count.
3. **Reduce OT Size and Bump Size** if you have simple scenes. This frees RAM for Lua and game data.
4. **Increase Bump Size** if triangles disappear at certain camera angles (buffer overflow).

### Skinned Meshes (RAM)

1. **Bone matrices are the biggest cost.** Each frame of each clip stores 24 bytes per bone. A 20-bone character with a 2-second walk at 15fps = ~14KB per clip.
2. **Lower Target FPS.** Sub-frame interpolation makes 15fps sampling look smooth. Only go to 30 for fast, precise animations.
3. **Use fewer bones.** 10-20 bones is typical for PS1 characters. Every bone adds 24 bytes × frame count to the scene data.
4. **Keep clips short.** Reuse idle/walk loops rather than long unique animations.
5. **Bone indices add up too.** Each triangle needs 3 bytes for bone assignments. A 500-triangle character = 1.5KB.

### Audio (SPU RAM)

1. **Lower sample rates.** 11025 Hz for sound effects, 22050 Hz for music.
2. **Keep clips short.** A 5-second clip at 22050 Hz is ~20KB ADPCM. Budget accordingly.
3. **24 voice limit.** Plan for worst-case simultaneous sounds.

### Lua (RAM + CPU)

1. **Prefer event-driven over onUpdate.** `onUpdate` runs every frame for every object that defines it.
2. **Only define callbacks you use.** The engine checks a bitmask - undefined callbacks have zero cost.
3. **Persist storage is limited to 16 entries.** Don't use it as a general-purpose database.

## OT Size and Bump Size Tuning

These are the two most important performance settings:

### OT Size

Controls depth sorting granularity. The ordering table has this many depth buckets. Triangles placed at the same depth bucket render in LIFO order.

- **Default: 8192**
- Larger = finer Z-sorting, less overlap artifacts
- Smaller = frees RAM, coarser depth sorting

### Bump Alloc Size

Per-frame GPU command buffer. Every rendered triangle and UI primitive consumes space.

- **Default: 129536**
- Larger = more primitives per frame, handles complex views
- Smaller = frees RAM, but primitives silently disappear when buffer fills

!!! warning "Silent triangle drops"
    When the bump allocator fills up, additional triangles are silently discarded. There's no error message. If geometry disappears at certain camera angles (where more is visible), increase this setting.
