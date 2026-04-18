# Known Issues

The following are known issues with the current version of SplashEdit and psxsplash. Pull requests are welcome for any of these - the project has grown significantly and contributions are greatly appreciated.

## Rendering

### Exterior scene BVH rendering may have issues
The BVH frustum culling system has been rewritten with a proper frustum extraction and AABB test, but may still have edge cases. **Interior scene type with rooms and portals is still recommended for best performance.**

### BVH preview visualization is broken
The BVH preview toggle in the Scene Exporter inspector does not display correctly. The Rooms/Portals preview works fine.

### Near-plane triangle artifacts
Triangles crossing the near plane are subdivided in 3D and re-projected. This replaced the previous Sutherland-Hodgman clipping approach. Rasterizer-limit triangles use vertex clamping instead of subdivision to avoid T-junction cracks. Minor visual artifacts may still appear on triangles very close to the camera.

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

### Navigation mesh plane error on terrain
On undulating terrain, some navigation regions may have high floor-plane error, causing the player to float above or sink below the actual surface. Use the **Terrain** preset or lower the **Merge Region Area** and **Max Edge Length** to produce smaller regions that track terrain height more accurately. The build statistics panel in the PSXPlayer inspector flags regions exceeding the Max Plane Error threshold.

### Rotating objects doesn't update collision bounds
Rotating objects via Lua does not recalculate their AABB (axis-aligned bounding box) collision bounds. The collision volume stays in the original orientation. Position changes via `Entity.SetPosition` now correctly shift the AABB and mark the object as dynamically moved for proper frustum culling.

### CD-DA audio requires ISO build
CD-DA playback only works from a disc image. It will not produce any audio when running via PCdrv (Emulator or Real Hardware build targets). Use the ISO build target.

## API Limitations

### Camera API overridden by navigation controller
In scenes with a PSXPlayer and navigation regions, the navigation controller continuously overrides camera position and rotation. Use `Camera.FollowPsxPlayer(false)` to take manual control of the camera from Lua. The Camera API is also usable during cutscenes, which temporarily suspend the nav controller.

### Camera.LookAt is incomplete
The `Camera.LookAt()` function exists in the Lua API but is a placeholder. It does not correctly point the camera at the target position.

### Persist storage limited to 16 entries
The cross-scene persistent data system supports only 16 key-value pairs. Exceeding this limit silently fails with no error.

### Memory report is very approximate
The memory preview in the Control Panel is very limited and approximate. Do not trust the numbers as accurate. A safe margin is when it shows around 50%.

## Platform

### Can get a little moody on Linux
Some editor functionality may behave inconsistently on Linux. The primary development and testing platform is Windows.

### macOS support is experimental
macOS (Apple Silicon) is now supported with automatic toolchain detection and PCSX-Redux downloads, but it has not been extensively tested. Build PATH detection for GUI apps has special handling, but edge cases may exist.

## Build Pipeline

### PCSX-Redux startup logs may be missed
When launching the emulator from the Control Panel, Unity can hang briefly during the process launch. Early console output from PCSX-Redux may be lost. This doesn't affect gameplay.

## Fonts

### Custom fonts can have character cropping issues
Individual characters in custom bitmap fonts may appear slightly cropped or cut off.

## Internal

### Controller rumble has no Lua API
The PS1 runtime supports DualShock rumble/vibration motors, and they can be driven via Rumble Small and Rumble Large tracks in [cutscenes](../components/cutscenes.md#controller-rumble) and [animations](../components/animations.md#controller-rumble). However, there is no Lua API for controlling rumble directly from scripts yet. This will be exposed in a future update.

### Default ISO license shows "Made with psxsplash"
When building an ISO without a custom license file, the default license displays "Made with psxsplash" on boot. Provide your own `.dat` license file in the build settings to override this.
