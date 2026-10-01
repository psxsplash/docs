# Sprites

2D sprites, drawn between the 3D scene and the UI. They are how you build a 2D
game on psxsplash, and how you add HUD elements, markers and effects to a 3D one.

Sprites are cheap: they are flat textured quads with no lighting, no GTE transform
and no depth sort against the world. The engine keeps a pool of **128** of them.

## Creating a sprite sheet

Right-click in the Project window: **Create > PSXSplash > Sprite Sheet**.
This makes a `PSXSpriteSheet` asset.

| Field | What it means |
|---|---|
| **Sheet Name** | the name Lua uses. Max 24 characters. |
| **Source Texture** | the atlas. Quantized and packed into VRAM alongside UI and 3D textures. |
| **Bit Depth** | 4-bit is cheapest and suits flat 2D art. |
| **Cell Width / Height** | one frame. The texture's dimensions should be whole multiples of these. |

Cells are numbered left to right, then top to bottom, starting at 0.

!!! warning "A sprite sheet is a cutout texture"
    Sprites are exported with alpha cutout on, so transparent pixels stay
    transparent. A 4-bit sheet has **fifteen** usable colours, not sixteen: entry 0
    is reserved for transparency. A sixteenth colour makes the quantizer start
    inventing dithered approximations of art that was drawn to be exact.

## Animations

Each sheet carries a list of animations defined over its own cells:

| Field | What it means |
|---|---|
| **Animation Name** | the name Lua uses. Max 24 characters. |
| **First Frame** | the cell the run starts at |
| **Frame Count** | how many consecutive cells it covers |
| **Frame Duration** | vsync frames each cell is held. At 60Hz, 6 gives 10fps |
| **Loop** | restart at the first frame, or hold the last |

Animations advance on a real-time accumulator, not once per rendered frame, so a
walk cycle plays at the same speed whatever the frame rate is doing.

## Adding sprites to a scene

Add a **PSX > PSX Sprite Sheet Ref** component (`PSXSprite`) to any GameObject in
the scene, one per sheet, and assign the sheet. Only referenced sheets are packed
into that scene's splashpack.

Sprite **instances** are not authored. That is the opposite of the UI, and
deliberate: a game spawns and destroys sprites as players join, bullets fire and
props die, and none of that is known at export time. What has to be authored is the
data, because that is what must be resident in VRAM.

So the sprites themselves are created at runtime from Lua:

```lua
function onSceneCreationEnd()
    local id = Sprite.Create("crew")
    Sprite.PlayAnim(id, "idle")
    Sprite.SetPos(id, 160, 120)
end
```

## Binding a sprite to an actor

The usual pattern for a character: create the sprite once, bind it, and never
touch its position again.

```lua
local id = Sprite.Create("crew")
Sprite.BindToActor(id, actor, 0, -8, -12)
```

The sprite follows the actor every frame with no per-frame Lua. Plane `0` is the
screen plane, where the actor's X/Z is the 2D ground plane and Y (height) is
ignored - a jumping actor should not slide up the screen. The offsets shift the
sprite from that point, usually up and left so it is centred on the actor's feet.

## Layers and the view offset

```lua
Sprite.SetLayer(id, 0)      -- 0 is frontmost
```

Layer decides draw order. Within a layer, order falls to creation order, so create
backgrounds before foregrounds and you will rarely need to think about it.

For a scrolling 2D game, move the world rather than every sprite:

```lua
Sprite.SetViewOffset(camX, camY)          -- shifts every sprite...
Sprite.SetIgnoreViewOffset(hudId, true)   -- ...except the pinned ones
```

## Facing

For a character with directional art, let the engine pick the animation:

```lua
Sprite.SetFacingFromYaw(id, "walk", 8, yaw)
```

Eight animations starting at `walk`, chosen from the actor's yaw. The current
frame and timer carry across a switch, so a character turning while walking does
not stutter back to frame 0.

## Limits and pitfalls

| Limit | Value |
|---|---|
| Sheets per scene | 16 |
| Animations per scene | 64 (across all sheets) |
| Live sprites | 128 |
| Sheet and animation name | 24 characters |

The first two are checked at export and reported with the offending sheet named, so
they fail in Unity rather than as an assert on the console.

!!! warning "The pool is 128 sprites, and failure is silent"
    `Sprite.Create` returns `-1` when the pool is full, and every other `Sprite.*`
    call ignores an invalid id without complaining. Check the id once at creation.
    Create your sprites at scene start and re-frame them rather than creating and
    destroying per frame.

!!! warning "There is no per-sprite alpha"
    `Sprite.SetColor` tints, it cannot fade. A "dimmed" overlay drawn as a
    translucent rectangle will simply be opaque. To darken part of the screen, draw
    a mask with a hole cut in it.

!!! tip "Sheet and animation names are checked at runtime, not at build"
    `Sprite.SheetIndex` and `Sprite.AnimIndex` return `-1` for a name the scene does
    not have, and `Sprite.PlayAnim` on a bad name does nothing at all - the sprite
    keeps its previous animation. Resolve names once at scene start and log the
    failures; a typo otherwise reads as a character that slides around in an idle
    pose.

## See also

- [Tilemaps](tilemaps.md) - a tileset is a sprite sheet
- [`Sprite` API reference](../lua/api-reference.md#sprite)
