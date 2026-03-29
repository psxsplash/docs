# SplashEdit Documentation

Build PlayStation 1 games using Unity as your scene editor.

---

!!! warning "Early Project Notice"
    SplashEdit has been a one-person project and it has grown significantly. Everything here should work, but rough edges are expected. Please [submit issues and pull requests](https://github.com/psxsplash/psxsplash/issues) if you run into problems. If you build something with SplashEdit, please share it on the [PSX.Dev](https://psx.dev) or Bandwidth Discord servers - it would mean a lot!

## What is SplashEdit?

SplashEdit is a Unity editor package that lets you build PlayStation 1 games using Unity as your level editor. You design scenes, place objects, set up UI, write Lua scripts for game logic, and SplashEdit exports everything into a binary format that runs on real PS1 hardware or in an emulator.

```
Unity Scene  ->  SplashEdit Export  ->  splashpack binary  ->  psxsplash PS1 runtime
```

- **SplashEdit** is the Unity package (editor tools, exporters, inspectors)
- **psxsplash** is the C++ PS1 runtime engine (built on [PSYQo](https://github.com/grumpycoders/pcsx-redux/tree/main/src/mips/psyqo)) that reads exported data and runs your game

You work entirely in Unity. SplashEdit handles texture quantization, mesh conversion, VRAM packing, Lua compilation, and binary serialization. Click "Build & Run" and it compiles the native PS1 code, packages your scenes, and launches the result.

!!! note "Credit"
    If you mention somewhere in your game that it was made with SplashEdit, that would make me very happy. It's not required, but it's appreciated!

## Features

- **Visual scene editing** in Unity with PS1-accurate preview
- **Automatic texture quantization** with Floyd-Steinberg dithering (4-bit, 8-bit, 16-bit)
- **VRAM packing** with deduplication
- **Lua scripting** for all game logic (event-driven architecture)
- **UI system** with canvases, text, images, progress bars, and custom fonts
- **Cutscene and animation** system with keyframed tracks and easing
- **Navigation mesh** generation via DotRecast
- **Room/portal occlusion** for interior scenes
- **Audio** conversion to PS1 SPU ADPCM format
- **Multi-scene** support with persistent data across scene loads
- **One-click build** to emulator, real hardware (via serial), or ISO disc image
- **Loading screens** with their own lightweight binary format

## Quick Links

<div class="grid cards" markdown>

-   :material-download:{ .lg .middle } **Getting Started**

    ---

    Install SplashEdit, set up your toolchain, and build your first scene.

    [:octicons-arrow-right-24: Installation](getting-started/installation.md)

-   :material-cube-outline:{ .lg .middle } **Scene Components**

    ---

    Everything you can put in a scene: objects, UI, audio, triggers, and more.

    [:octicons-arrow-right-24: Components](components/index.md)

-   :material-script-text:{ .lg .middle } **Lua Scripting**

    ---

    Write game logic with the full Lua API reference and working patterns.

    [:octicons-arrow-right-24: Lua Scripting](lua/index.md)

-   :material-school:{ .lg .middle } **Tutorials**

    ---

    Step-by-step guides for common game mechanics.

    [:octicons-arrow-right-24: Tutorials](tutorials/index.md)

</div>
