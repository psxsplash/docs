# Building & Running

There is no separate build step. The editor exports your scene in the background a moment after each edit, so the **status bar** is always showing the real budgets for what would ship: VRAM, SPU RAM, main RAM and triangles. **Play**, **Run on hardware** and **Export** use that same export.

| You want | Press | Where it runs |
|----------|-------|---------------|
| To try the scene now | **Play** (**F5**) | pcsx-redux inside the editor |
| To test on a console | **Run on hardware** (**Ctrl+F5**) | A PS1 with Unirom, over a serial cable |
| To hand it to someone | **Export** (**Ctrl+B**) | A `.bin` + `.cue` disc image |

The details are in [Play, Hardware and Discs](../getting-started/play-and-export.md).

## What an export writes

For each scene in the [build list](scene-settings.md#build-scenes), the editor writes files named `scene_<i>`:

| File | Holds |
|------|-------|
| `.splashpack` | Objects, meshes, collision, navigation, UI, cutscenes, animations and the Lua scripts, compiled |
| `.vram` | Textures and palettes, packed into VRAM |
| `.spu` | Audio clips, as SPU-ADPCM |
| `.geo` | Streamed level geometry, when [streaming](world-streaming.md) is on |
| `.loading` | The [loading screen](scene-settings.md#loading-screen), when there is one |

Block scripts are turned into Lua at this point. Lua is checked by the same compiler as the [Lua editor](lua-editor.md) uses.

## Problems

The right of the status bar reads **No problems**, or the number of errors and warnings. Click it for the list.

- **Errors** stop the export. Play and Export tell you what to fix: a missing file, a track that targets an object that is not there, a portal with no Room, a script that does not compile.
- **Warnings** do not stop anything. They flag things like a texture that could be 4 bpp, a model with no vertex colours or a long skinned clip.

## Budgets

The meters are the console's limits:

| Meter | Limit |
|-------|-------|
| **VRAM** | 1024 KB |
| **SPU RAM** | 512 KB |
| **RAM** | What main RAM leaves for the scene after the engine |

When one turns red, see [Memory & Performance](../reference/memory.md).

## Debug overlays

**Build > Performance Overlay** and **Build > Room Debug Overlay** boot a psxsplash build that draws an FPS counter and heap bar, or the room wireframe. See [Debug overlays](../getting-started/play-and-export.md#debug-overlays).
