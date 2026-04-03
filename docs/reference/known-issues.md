# Known Issues

The following are known issues with the current version of SplashEdit and psxsplash. Pull requests are welcome for any of these - the project has grown significantly and contributions are greatly appreciated.

## Rendering

### Exterior scene BVH rendering may have issues
The BVH frustum culling system has been rewritten with a proper frustum extraction and AABB test, but may still have edge cases. **Interior scene type with rooms and portals is still recommended for best performance.**

### BVH preview visualization is broken
The BVH preview toggle in the Scene Exporter inspector does not display correctly. The Rooms/Portals preview works fine.

### Near-plane triangle artifacts
Triangles crossing the near plane are subdivided in 3D and re-projected. Rasterizer-limit triangles use vertex clamping instead of subdivision to avoid T-junction cracks. Minor visual artifacts may still appear on triangles very close to the camera.

### Fog edge cases
Distance fog uses a Silent Hill-style two-pass approach (per-vertex fog blending with additive overlay). Most configurations work well, but extreme density values or very close geometry may produce visual artifacts.

### PSX UI images overblown with white tint
Using a full white tint (255, 255, 255) on `PSXUIImage` elements causes washed-out, overblown colors. Use a tint around 128, 128, 128 for normal appearance.

### Portal visibility at close range
Portal projection has been improved (int16 overflow fix, conservative fullscreen rect for near-camera portals), but edge cases may remain when standing exactly on a portal boundary.

## Scene Setup

### Interior scenes need at least one room defined
If you set the scene type to Interior but don't add any `PSXRoom` components, rendering glitches will occur. Always define at least one room volume.

### All geometry must be enclosed in rooms
In interior scenes, geometry not contained within any `PSXRoom` volume goes to a catch-all bucket that's always rendered. Make sure room volumes fully enclose all visible geometry.

### Textures must be power-of-two
All textures must be power-of-two in both width and height, with a maximum size of 256x256. SplashEdit does NOT validate this - incorrect textures produce broken results silently.

## Physics & Navigation

### Jumping is visual only
Player jumping currently has no collision interaction with the environment. The player moves up and down visually but does not interact with platforms or ceilings.

### Nav Cell Height setting is broken
The Nav Cell Height parameter on PSXPlayer does not work correctly. Leave it at the default value.

### Rotating objects doesn't update collision bounds
Rotating objects via Lua does not recalculate their AABB (axis-aligned bounding box) collision bounds. The collision volume stays in the original orientation.

## API Limitations

### Camera API overridden by navigation controller
In scenes with a PSXPlayer and navigation regions, the navigation controller continuously overrides camera position and rotation. The Camera API is primarily useful during cutscenes.

### Camera.LookAt is incomplete
The `Camera.LookAt()` function exists in the Lua API but is a placeholder. It does not correctly point the camera at the target position.

### Persist storage limited to 16 entries
The cross-scene persistent data system supports only 16 key-value pairs. Exceeding this limit silently fails with no error.

### Memory report is very approximate
The memory preview in the Control Panel is very limited and approximate. Do not trust the numbers as accurate. A safe margin is when it shows around 50%.

## Platform

### Can get a little moody on Linux
Some editor functionality may behave inconsistently on Linux. The primary development and testing platform is Windows.

### No macOS support
SplashEdit does not support macOS. The developer does not have a Mac to develop and test on.

## Build Pipeline

### PCSX-Redux startup logs may be missed
When launching the emulator from the Control Panel, Unity can hang briefly during the process launch. Early console output from PCSX-Redux may be lost. This doesn't affect gameplay.

## Fonts

### Custom fonts can have character cropping issues
Individual characters in custom bitmap fonts may appear slightly cropped or cut off.

## Internal

### Navigation code is a mess
The navigation mesh generation system works but the internal code quality needs improvement. A rewrite is planned.

### Default ISO license shows "Made with psxsplash"
When building an ISO without a custom license file, the default license displays "Made with psxsplash" on boot. Provide your own `.dat` license file in the build settings to override this.
