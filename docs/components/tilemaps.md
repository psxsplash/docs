# Tilemaps

A grid-based 2D level: painted tiles, per-tile walkability, and objects placed on
the grid. The engine draws it, answers collision queries against it, and moves
actors through it, so a 2D game needs no per-tile Lua at all.

## Creating a tilemap

Right-click in the Project window: **Create > PSXSplash > Tilemap**.

| Field | What it means |
|---|---|
| **Tileset** | a [sprite sheet](sprites.md). Its cell size becomes the tile size. |
| **Width / Height** | the grid, in tiles |
| **Brushes** | the palette you paint with |
| **Object Kind Names** | labels for the kind bytes, for the editor only |

!!! tip "A tileset is just a sprite sheet"
    There is no separate tileset asset type. That means tiles animate, tint and
    flip exactly like sprites do, and the packer treats them the same way.

## Brushes

A brush is one entry in the palette, and it carries more than a picture:

| Field | What it means |
|---|---|
| **Name** | for the editor |
| **Tileset Cell** | which cell of the tileset to draw |
| **Walkable** | whether actors can enter this tile |
| **Object Kind** | a byte the game defines. 0 for plain scenery. |
| **Object Id** | distinguishes instances of the same kind. -1 for none. |
| **Editor Color** | palette tint, editor only |

**Walkability is a property of the brush, not a separate layer.** Paint a wall and
it is solid; there is no second pass to keep in sync, and no way for the collision
map to drift out of step with what the player can see.

## Painting

Select the tilemap asset and paint in the inspector. Left-click paints the selected
brush, right-click erases.

## Adding it to a scene

Add **PSXSplash > PSX Tilemap Renderer** to a GameObject and assign the tilemap
asset. One per scene.

The tileset needs no **PSX Sprite Sheet Ref** of its own: the exporter pulls it
into the same sheet table automatically, and packs it once whether or not a
`PSXSprite` also points at it. It counts against the
[16-sheet limit](sprites.md#limits-and-pitfalls) like any other.

## Seeing it in the scene view

The painted map is drawn in the 3D viewport, on the **XZ plane**, where the
engine actually puts it: tile (0,0) is world (0,0) and **one world unit is one
pixel**. Everything else you place — spawn points, actors, props — is positioned
against that, so being able to see it is the difference between laying out a
level and guessing at one.

The renderer's GameObject transform is **ignored on export**, and the preview
draws at the origin for the same reason: a preview that followed the transform
would lie about where the floor is the moment somebody nudged the object.

Select the tilemap renderer to also get the grid, a red wash over the **solid**
cells and a ring on every **object** tile — walkability and gameplay tags are
invisible in the art, and they are what a layout is actually about.

Toggle the whole thing from `PlayStation 1 > Show Tilemap in Scene View`, or from
the button on the renderer's inspector. The preview mesh is cached and rebuilt
only when the map changes, so a 64x64 map costs nothing to leave on; the Tile
Painter refreshes it as you paint.

To see the map **with the UI over it**, at PS1 pixels, use the **PSX Screen**
overlay — see [UI](ui.md#seeing-it-the-psx-screen-overlay).

## Collision and movement

The engine owns movement, so a 2D character controller is a few lines:

```lua
function onUpdate(self, dt)
    local dx, dz = 0, 0
    if Input.IsHeldPlayer1(Input.LEFT)  then dx = -SPEED end
    if Input.IsHeldPlayer1(Input.RIGHT) then dx =  SPEED end
    if Input.IsHeldPlayer1(Input.UP)    then dz = -SPEED end
    if Input.IsHeldPlayer1(Input.DOWN)  then dz =  SPEED end

    Tile.MoveActor(me, dx, dz)
end
```

`Tile.MoveActor` tests each axis separately, so running into a wall at an angle
slides along it instead of stopping dead.

You can also query directly:

```lua
Tile.Walkable(x, z)              -- can something stand here?
Tile.RayClear(x0, z0, x1, z1)    -- is there line of sight?
```

`RayClear` is what you want for enemy vision, or for hiding things the player
cannot see.

!!! warning "Movement deltas are whole pixels"
    `dx` and `dz` are truncated to integers, which has two consequences worth
    knowing before you debug them:

    - a speed below 1 pixel per frame rounds to **zero** and the actor never moves;
    - scaling a speed of 2 by 0.707 for a diagonal gives **1**, a 50% cut rather
      than the 29% that normalising a diagonal actually calls for.

    Accumulate in sub-pixels and spend whole ones:

    ```lua
    local SUB = 256
    accX = accX + vx                        -- vx in 1/256 pixels per frame
    local dx = (accX - accX % SUB) / SUB
    accX = accX - dx * SUB
    ```

!!! tip "A scene with no tilemap reports everything walkable"
    `Tile.Walkable` and `Tile.RayClear` answer `true` when there is no map, so a
    movement script written against them keeps working unchanged in scenes that
    have none. `Tile.Active()` tells you which case you are in.

## Objects on the grid

Anything you paint with a non-zero **Object Kind** is exported as a placed object,
which is how you position doors, pickups, spawn points and stations without
authoring a GameObject for each:

```lua
for i = 1, Tile.ObjectCount() do
    local kind, id, x, z = Tile.ObjectAt(i)
    if kind == MY_DOOR then
        doors[id] = { x = x, y = z }
    end
end
```

**The engine attaches no meaning to the kind byte.** Define your own constants and
make sure they match what the map was painted with; there is no shared enum to get
out of step, and equally nothing to stop you painting kind 7 and reading kind 8.

## See also

- [Sprites](sprites.md)
- [`Tile` API reference](../lua/api-reference.md#tile)
