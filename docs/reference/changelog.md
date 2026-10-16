# Changelog

## Nightly editor

The editor, engine and emulator ship as one download, rebuilt from every change to the editor's `main` branch. [Download](https://github.com/psxsplash/splashedit-ng/releases/tag/nightly).

- A standalone editor for Windows, Linux and macOS (Apple silicon) with a PS1-accurate viewport, scene tree, inspector, project browser and live VRAM, SPU RAM and main RAM meters.
- **Play** boots the scene in the bundled pcsx-redux next to the viewport and reloads it as you edit. **Run on hardware** sends it to a console over serial with Unirom. **Export** writes a `.bin` and `.cue` disc image.
- A [Lua editor](../editor/lua-editor.md) with highlighting, a compile check, autocomplete with signatures, go to definition and find and replace.
- A [block editor](../editor/block-editor.md) that makes Lua from stacks of blocks. Blocks and Lua can be mixed in one scene.
- [UI canvases](../editor/ui-canvas.md), [cutscenes and animations](../editor/timeline.md) on a timeline, [skinned animation](../editor/skinned-animation.md), [rooms and portals](../editor/navigation-rooms.md), [audio clips](../editor/audio.md), [runtime point lights](../editor/lighting.md) and [world streaming](../editor/world-streaming.md).
- Model import for glTF, FBX, OBJ and Blender files, with per-model import settings and a texture bit depth check that offers 4 bpp.
- Memory card save settings, disc volume label and licence options, and a loading screen canvas.
- Lua: `Light` functions, `SkinnedAnim.GetBone`, and `Camera.LookAt` now points the camera at its target.

## 2.4.0 and earlier

The Unity-based editor, SplashEdit. Its documentation and changelog stay online at [psxsplash.github.io/docs/unity](https://psxsplash.github.io/docs/unity/).
