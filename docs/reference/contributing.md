# Contributing

PSXSplash and SplashEdit are open-source projects. The codebase has grown significantly as a one-person project and **pull requests are very welcome**.

!!! note "Share your creations!"
    If you build something with SplashEdit, please share it on the [PSX.Dev](https://psx.dev) or Bandwidth Discord servers. It would mean a lot to see what people make with this tool.

## Repositories

| Repository | Contents |
|-----------|----------|
| [psxsplash](https://github.com/psxsplash/psxsplash) | C++ PS1 runtime engine (built on PSYQo) |
| [splashedit](https://github.com/psxsplash/splashedit) | Unity editor package (C# exporters, inspectors, build pipeline) |

## Areas Where Help Is Needed

### High Priority

- **Exterior BVH rendering** - The frustum culling system for exterior scenes needs a complete rewrite. Interior room/portal culling works well, but the BVH path is broken.
- **Navigation code cleanup** - The nav mesh generation works but the code needs refactoring for maintainability.
- **Fog implementation** - Distance fog is partially implemented but has visual bugs.
- **Nav Cell Height fix** - The setting is broken and needs debugging.

### Medium Priority

- **Polygon subdivision stitching** - Seam artifacts appear between subdivided polygons.
- **Camera.LookAt implementation** - Needs proper atan2 math to correctly orient the camera.
- **Camera.GetRotation** - Rotation decomposition from the internal matrix.
- **Jump collision** - Jumping is currently visual only with no environment interaction.
- **Dynamic collision AABB updates** - Rotated objects don't update their collision bounds.
- **Portal close-range visibility** - Portals don't render the connected room when standing very close.

### Nice to Have

- **macOS support** - The developer doesn't have a Mac, so contributions here are especially welcome.
- **Texture size validation** - SplashEdit should warn when textures aren't power-of-two or exceed 256x256.
- **Additional Lua API functions** - New convenience functions, math utilities, or system capabilities.
- **Bug fixes** - Any bug fix, no matter how small, is appreciated.
- **Documentation improvements** - Corrections, clarifications, additional examples.

## Getting Started

1. Fork the repository
2. Clone your fork
3. Set up the development environment (see [Installation](../getting-started/installation.md))
4. Make your changes
5. Test thoroughly
6. Submit a pull request
