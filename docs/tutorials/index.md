# Tutorials

Step-by-step guides for building common game mechanics with SplashEdit. Each tutorial assumes you have a working scene (see [Quick Start](../getting-started/quick-start.md)).

## Available Tutorials

| Tutorial | What You'll Build |
|----------|------------------|
| [Collectible Pickups](collectibles.md) | Objects the player walks into to collect, with score tracking and sound |
| [NPC Dialogue](npc-dialogue.md) | An interactive NPC with multi-line dialogue and button advancement |
| [Animated Doors](animated-doors.md) | A door that opens and closes with animation and sound |
| [Scene Transitions](scene-transitions.md) | Portals that transport the player between scenes |
| [Health & Damage](health-damage.md) | Hazard zones, healing zones, and a color-coded health bar |

## Prerequisites

Each tutorial builds on these basics:

- A scene with a `PSXSceneExporter`, `PSXPlayer`, and a floor
- The scene added to the Control Panel's scene list
- Familiarity with [fixed-point math](../lua/fixed-point.md) (no float literals)

## Code Style Notes

All tutorial code follows these conventions:

- **Event-driven** - everything responds to events, no per-frame loops
- **No float literals** - use `FixedPoint.new()` for all fractional values
- **Scene as controller** - shared functions defined in scene.lua and published to `_G`
- **One-time flags** - local booleans prevent repeated actions
