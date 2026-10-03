# World Streaming

World Streaming lets a scene be bigger than the PS1's RAM. The static level geometry is split into regions that are loaded off the CD as the camera moves, and dropped again when it moves away.

!!! info "Not in 2.4.0"
    World Streaming is on the `main` branch of psxsplash and SplashEdit. It is not in 2.4.0.

## What Streams

Only static level geometry streams. Textures, collision, navigation, scripts and sounds always stay loaded.

!!! note
    Textures never stream. Uploading to VRAM is too slow to do during play.

## Turning It On

Tick **Stream World Geometry** on the [Scene Exporter](scene-exporter.md). Everything else is automatic.

Enable **Show in Scene View** to see what the exporter decided:

- Region outlines, labelled with their size in KB
- Objects that stay always loaded, drawn in **orange**
- The load and unload circles around the selection (or the scene camera if nothing is selected)

### Advanced

| Field | Default | Description |
|-------|---------|-------------|
| Region Size | 0 (automatic) | Size of one region. Automatic picks the size that uses the least streaming memory. |
| Load Distance | 0 (automatic) | How close the camera must be for a region to load. Automatic is how far the console draws (the ordering-table depth, or the fog distance if fog is on) plus 25%. |

Regions unload at 1.5x the load distance.

## Which Objects Stream

An object streams only if all of these are true:

- It has triangles
- It has no Lua script
- It is not skinned
- It is not an interactable or an agent
- It is not a dynamic collider
- It is not moved by an animation or cutscene track
- Its name does not appear in any Lua script

Everything else stays loaded.

!!! danger "Objects reached by index"
    The exporter finds script references by name. A script that reaches an object only by index (`Entity.FindByIndex`) cannot be detected, and the object may be unloaded when the script needs it. Give such objects a script, or look them up by name.

## Inspector and Memory Report

With streaming on, the Scene Exporter inspector shows:

- Region count
- Streamed and always-loaded object counts
- Streaming memory
- RAM saved

The [Control Panel](../getting-started/control-panel.md) memory report counts streaming memory in the RAM bar and the `.geo` file on the CD.

## Warnings

| Warning | What to do |
|---------|-----------|
| Nothing can stream | Every object is excluded by the rules above. Turn streaming off, or free up some static geometry. |
| A region is over 256 KB | Lower Region Size, or split the large object. |
| One region is much bigger than the rest | Every streaming slot costs as much as the biggest region. Split the named object, or lower Region Size. |
| More regions in range than the 32 slots | Lower Load Distance. |
| Streaming saves no RAM | Leave streaming off for this scene. |
| A script calls `Audio.PlayCDDA` | See below. |

## CD-DA Music

!!! warning "No CD-DA in a streaming scene"
    The CD drive is busy reading the level, so `Audio.PlayCDDA` is ignored in a streaming scene. Use SPU [audio clips](audio.md) instead.

## Output

Streaming adds a file `scene_N.geo` (`SCENE_N.GEO` on the disc) next to the scene's other files. It is added to the ISO automatically.

Streaming works in the emulator (PCdrv) and on CD.
