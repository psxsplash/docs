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

- **Navigation code cleanup** - The nav mesh generation works but the code needs refactoring for maintainability.
- **Nav Cell Height fix** - The setting is broken and needs debugging.

### Medium Priority

- **Camera.LookAt implementation** - Needs proper atan2 math to correctly orient the camera.
- **Camera.GetRotation** - Rotation decomposition from the internal matrix.
- **Jump collision** - Jumping is currently visual only with no environment interaction.
- **Dynamic collision AABB updates** - Rotated objects don't update their collision bounds.

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

## Documentation Versioning

The docs use [mike](https://github.com/jimporter/mike) for versioning. A permanent version dropdown appears in the site header, letting visitors switch between versions.

### Branch Strategy

| Branch | Purpose |
|--------|---------|
| `main` | Current released version (deployed to the live site) |
| `dev` | Next version in development (not deployed, not public) |

### Contributing Docs for the Next Version

If your changes apply to an **unreleased** version:

1. Base your branch off `dev` (not `main`)
2. Submit your PR targeting the `dev` branch
3. Your changes will stay invisible to the public until the version is released

If your changes apply to the **current** version (typo fixes, corrections, clarifications):

1. Base your branch off `main`
2. Submit your PR targeting `main`
3. Changes go live automatically on merge

### Releasing a New Version (Maintainers)

When a new version is ready to go public:

1. Update the `VERSION` file on the `dev` branch to the new version number (e.g., `2.1`)
2. Merge `dev` into `main`
3. Push to `main` — the CI will automatically deploy the new version and set it as `latest`
4. Optionally tag the release: `git tag v2.1 && git push --tags`
5. Recreate the `dev` branch from `main` for the next development cycle

Previous versions remain accessible through the version dropdown. Visitors are always routed to `latest` by default.

### Local Preview

```bash
# Preview the docs locally (without versioning)
mkdocs serve

# Preview how versioned docs look locally
mike deploy 2.0 latest
mike serve
```
