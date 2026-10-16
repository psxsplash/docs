# World Streaming

A scene can be bigger than the PS1's RAM. With **Stream world** on, the static level geometry is split into regions that load from the disc as the camera moves and drop again when it moves away. Textures, collision, navigation, scripts and sounds always stay loaded; only triangles stream.

## Turning it on

Select nothing, and under **World streaming** switch on **Stream world**. Everything else is automatic. The section then reports, once the scene exports:

- the region count and size, and how many objects stream (and how many were cut at region edges),
- how far from the camera scenery loads,
- the slots the engine keeps, and the RAM they cost while playing,
- the RAM saved.

## Which objects stream

An object streams only if all of these are true:

- It has triangles.
- It has no script.
- It is not skinned.
- It is not an Interactable.
- It is not a dynamic collider.
- No animation or cutscene track moves it.
- Its name does not appear in any Lua script.

Everything else stays loaded. Large meshes are cut at region edges.

!!! warning "Objects reached by index"
    The exporter finds script references by name. A script that reaches an object only by index (`Entity.FindByIndex`) cannot be detected, and the object may be unloaded when the script needs it. Look such objects up by name, or give them a script.

## Warnings

| Warning | What to do |
|---------|------------|
| Nothing can stream | Every object is excluded by the rules above. Take scripts off scenery that does not need one, or turn streaming off. |
| Too many pieces to stream | The world splits into more than 65535 pieces and exports without streaming. Merge small props. |
| A region is slow to load | The biggest region is large. **Shorten draw distance** lets regions be smaller. |
| One area is much denser | Every streaming slot is as big as the biggest region. Spread that area's detail out, or simplify its meshes. |
| Far scenery may appear late | More regions are near the camera than the engine keeps. **Shorten draw distance**. |
| Streaming does not save RAM here | Shorten the draw distance, or turn streaming off for this scene. |
| CD music will not play | A script calls `Audio.PlayCDDA`, and a streamed scene keeps the disc drive busy. Use [audio clips](audio.md). |

The draw distance is how far the console draws: the [ordering table](scene-settings.md#render-buffers), or the fog wall if fog is on.

## Output

Streaming adds a `scene_N.geo` file next to the scene's other exported files, and the disc image carries it as `SCENE_N.GEO`. It works in Play (PCdrv), on hardware and on a disc.
