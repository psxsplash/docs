# Building & Running

## Build Pipeline

When you click **BUILD & RUN**, SplashEdit runs this pipeline:

1. **Lua Compilation** - All Lua scripts are extracted and compiled to bytecode using a PS1-based Lua compiler (luac_psx). This runs headlessly in PCSX-Redux.
2. **Scene Export** - Each scene in the scene list is exported to a `.splashpack` binary file. Loading screens are exported as `.loading` files.
3. **Manifest Generation** - A `manifest.bin` listing all scenes is written.
4. **Native Compilation** - The psxsplash C++ runtime is compiled with your configuration.
5. **Launch** - The compiled executable and data are sent to the target.

## Build Targets

### Emulator (PCSX-Redux)

- Launches PCSX-Redux with the compiled PS-EXE
- Scene files served via PCdrv (virtual filesystem)
- Console output visible in the PSX Console window
- Best for development and testing

### Real Hardware

- Uploads the PS-EXE to a real PS1 via Unirom and serial
- Files served via PCdrv over serial
- Requires a serial connection and Unirom-compatible setup (DEBG kernel)

### ISO

- Generates a `.bin/.cue` disc image with SYSTEM.CNF
- Includes the executable, all splashpacks, loading packs, and manifest
- Ready to burn to disc or load in any emulator
- For distribution

!!! note "Default license file"
    If you don't provide a custom license file, the ISO uses a built-in modified license that displays **"Made with psxsplash"** on boot. You can replace this with your own `.dat` license file in the build settings.

## Build Configuration

| Setting | Effect |
|---------|--------|
| Debug mode | Includes debug symbols, less optimization |
| Release mode | Full optimization, smaller binary |
| Clean Build | Runs `make clean` first (slower but ensures clean state) |
| Memory Overlay | Shows RAM usage bar at runtime |
| FPS Overlay | Shows frame counter at runtime |
| Room Debug Overlay | Renders all room triangles in per-room colors for culling diagnosis |

## OT Size and Bump Alloc Size

These two settings control the trade-off between rendering capacity and memory:

### OT Size (Ordering Table)

The ordering table determines depth sorting resolution. Each entry is a depth bucket for polygon sorting. The PS1 GPU draws back-to-front using this table.

- **Larger** = finer Z-sorting, less Z-fighting, but more RAM
- **Smaller** = coarser sorting, possible visual overlap, but frees RAM
- Default: 8192

### Bump Alloc Size

The per-frame GPU command buffer. Every triangle, line, and UI element uses space here.

- **Larger** = more visible polygons per frame
- **Smaller** = frees RAM for Lua and game data, but triangles silently disappear when the buffer fills

!!! warning
    If triangles disappear at certain camera angles (where more geometry is visible), your bump allocator is probably too small. Increase this setting.

Default: 129536

## Build Output

The build process creates files in `[Unity Project]/PSXBuild/`:

| File | Purpose |
|------|---------|
| `scene_0.splashpack` | Scene 0 data |
| `scene_0.loading` | Scene 0 loading screen (if assigned) |
| `manifest.bin` | Scene list metadata |
| `psxsplash.ps-exe` | Compiled PS1 executable |
| `lua_src/` | Extracted Lua source files |
| `lua_compiled/` | Compiled Lua bytecode |
| `build.log` | Make output (on failure) |
| `psxsplash.bin/.cue` | ISO output (ISO target only) |
