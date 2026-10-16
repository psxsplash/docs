# Tutorials

Step-by-step builds for common game mechanics. Each starts from the courtyard example (see [Your First Project](../getting-started/first-project.md)) or any scene with a floor and a Player, and each gives the logic as Lua you can paste into the [Lua editor](../editor/lua-editor.md). The same logic can be built from [blocks](../editor/block-editor.md).

| Tutorial | What you build |
|----------|----------------|
| [Collectible Pickups](collectibles.md) | Objects the player walks into, with a score on the HUD and a sound |
| [Animated Doors](animated-doors.md) | A door that opens and closes on a button press, with an animation from the timeline |
| [Scene Transitions](scene-transitions.md) | A trigger that moves the player to another scene |
| [Health & Damage](health-damage.md) | Hazard and healing zones and a colour-changing health bar |

## Before you start

- A scene with a **Player** and floors with a **Static** collider, so Play lets you walk.
- [Fixed-point math](../lua/fixed-point.md): there are no decimal literals.

## Conventions

- **Event-driven.** Everything responds to events; nothing polls in `onUpdate`.
- **The scene script as controller.** Shared functions live in the scene script and are published to `_G`, so object scripts can call them. See [Lua scripting](../lua/index.md#script-environments).
- **One-time flags.** A local boolean stops an action from repeating.
