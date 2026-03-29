# Rooms & Portals (Interior Scenes)

Interior scenes use a room/portal system for occlusion culling. Only rooms visible through portal connections from the camera's current room are rendered. This is much more efficient than rendering everything.

## When to Use Interior Mode

Set the scene type to **Interior** on the [Scene Exporter](scene-exporter.md) when your scene is an enclosed space: buildings, dungeons, hallways, caves, or any area with walls separating distinct rooms.

!!! tip
    Interior mode with room/portal culling performs much better than exterior BVH mode. Use it whenever your scene has walls or separations between areas.

## Setting Up Rooms

### PSXRoom Component

1. Create an **empty GameObject** where you want a room
2. Add the `PSXRoom` component
3. Adjust **Volume Size** to cover the entire room
4. Use **Volume Offset** if the center doesn't align with the transform position

| Field | Description |
|-------|-------------|
| Room Name | Display name shown in editor gizmos |
| Volume Size | Box dimensions (width, height, depth) |
| Volume Offset | Center offset from the transform position |

Rooms appear as **green wireframe boxes** in the Scene view.

### Room Rules

!!! danger "All geometry must be enclosed in rooms"
    Every piece of visible geometry should be inside at least one room volume. Geometry outside all rooms goes to a catch-all bucket that's always rendered, defeating the purpose of occlusion culling. Make your room volumes large enough to fully contain all walls, floors, and ceilings.

- Every interior scene **must have at least one room**
- Rooms should fully enclose all geometry they contain
- Rooms can overlap slightly (triangles are assigned to the best-fit room)

## Setting Up Portals

### PSXPortalLink Component

1. Create an **empty GameObject** at a doorway or opening between two rooms
2. Add the `PSXPortalLink` component
3. Set **Room A** and **Room B** to the two rooms this portal connects
4. Set **Portal Size** to the width and height of the opening
5. Position and rotate the portal to face the doorway

| Field | Description |
|-------|-------------|
| Room A | First connected room |
| Room B | Second connected room |
| Portal Size | Width and height of the opening |

Portals appear as **orange rectangles** in the Scene view.

### Portal Rules

- Portals must connect exactly two rooms
- Position the portal at the actual doorway/opening
- Portal size should match the opening dimensions
- The portal's orientation (rotation) matters for visibility calculation

## How It Works at Runtime

1. The runtime determines which room the camera is in (point-in-AABB test)
2. Starting from the camera's room, it checks which portals are visible on screen
3. For each visible portal, the connected room is added to the render list
4. This continues recursively through visible portals (with screen-space clipping)
5. Only rooms on the render list have their triangles drawn

## Triangle Assignment

During export, triangles are automatically assigned to rooms:

- Each triangle's vertices are tested against room AABBs
- The room containing the most vertices wins
- Ties are broken by which room center is closest
- Triangles at room boundaries (doorways, edges) are **duplicated into both adjacent rooms** to prevent gaps

!!! note
    You don't need to manually assign geometry to rooms. Just make sure your room volumes fully enclose all the geometry.

## Known Issues

!!! bug "Portal visibility at close range"
    When standing directly in front of a portal, the room it connects to may not render correctly. This is a known issue with the portal screen-rect calculation at very close distances.
