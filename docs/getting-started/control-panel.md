# The SplashEdit Control Panel

The Control Panel (**PlayStation 1 -> SplashEdit Control Panel**, or ++ctrl+shift+l++) is your central hub for everything: dependencies, scene management, and building.

## Dependencies Tab

Manages the native psxsplash project and toolchain. See [Installation](installation.md) for the full setup walkthrough.

### Native Project

- **Download from GitHub** - Select a release version from the dropdown, click Clone
- **Manual path** - Browse to a local psxsplash source directory (must contain a Makefile)
- Shows current version when installed

### Toolchain

Five status rows showing each required tool with Install/Download buttons. Each shows a green "Ready" badge when detected.

## Scenes Tab

Your project's scene list. Scenes are exported in order - **Scene 0 is loaded first** at runtime.

| Action | How |
|--------|-----|
| Add current scene | Click **"+ Add Current Scene"** |
| Add any scene | Click **"+ Add Scene..."** and browse |
| Drag and drop | Drag SceneAsset files from the Project window |
| Reorder | Use the arrow buttons (scene index matters for `Scene.Load()`) |
| Remove | Click the **X** button |

!!! important
    Scene indices are assigned by position in this list. If you reorder scenes, any `Scene.Load(index)` calls in your Lua scripts need to match the new order.

## Build Tab

Build configuration and the main build button. See [Building & Running](building.md) for full details.

### Configuration

| Setting | Description | Default |
|---------|-------------|---------|
| Target | Emulator, Real Hardware, or ISO | Emulator |
| Mode | Debug or Release | Debug |
| Clean Build | Full rebuild from scratch | On |
| Memory Overlay | Show heap/RAM usage bar at runtime | Off |
| FPS Overlay | Show frame rate counter at runtime | Off |
| Room Debug Overlay | Render all room triangles in per-room colors for culling diagnosis | Off |
| OT Size | Ordering table size (depth sorting precision) | 8192 |
| Bump Alloc Size | Per-frame GPU primitive buffer size | 129536 |

### Target-Specific Options

- **Real Hardware**: Serial port name and baud rate for Unirom upload
- **ISO**: Volume label (max 31 chars, uppercase) and optional license `.dat` file

### Buttons

- **BUILD & RUN** - Export all scenes, compile, and launch
- **STOP EMULATOR / STOP PCdrv HOST** - Kill the running instance
- **Export Only** - Export scenes without compiling
- **Compile Only** - Compile without re-exporting

### Memory Report

After exporting, a foldout shows per-scene memory breakdown:

- **Main RAM** usage (ordering table, bump allocator, scene data)
- **VRAM** usage (framebuffer, texture atlases, CLUTs)
- **SPU RAM** (audio clips)
- **CD Storage** (splashpack size, loader pack size)

!!! danger "Memory Report is very approximate"
    The memory preview in the Control Panel is **very limited and very approximate**. Do not trust it as accurate. A safe rule of thumb: if it shows **50% usage**, you're probably fine. If it shows higher, you may already be at the limit. Always test on real hardware or emulator to verify.
