# Glossary

Terms used in these docs that assume PS1 background.

### OT (Ordering Table)

A fixed-size array of depth buckets the GPU primitives are linked into before they are sent to the GPU. The PS1 has no Z-buffer, so this is how depth sorting works. The slot count is also how far the camera sees. See [Render buffers](../editor/scene-settings.md#render-buffers).

### GTE (Geometry Transformation Engine)

The PS1 CPU's coprocessor 2, a fixed-function unit that does matrix and vector math (transforms, projection, lighting) in hardware. Its coordinates are 16-bit fixed point, which is why exported positions are divided by the scene's **GTE scaling** and must sit within 8 x the scaling of the origin. See [Rendering](../editor/scene-settings.md#rendering).

### Fixed-point

Numbers stored as integers with an implied binary point. psxsplash Lua uses 20.12: 4096 is 1.0. See [Fixed-Point Math](../lua/fixed-point.md).

### Splashpack

The binary file the editor exports a scene into (`scene_0.splashpack`). The psxsplash engine reads it to load a scene's objects, meshes, navigation, UI and scripts. Textures go in a `.vram` file beside it and audio in a `.spu` file.

### Bump allocator

A per-frame allocator that resets every frame, used for the GPU command buffer. The editor calls it the **Primitive buffer**.

### VRAM

The GPU's own 1 MB, addressed as 1024 x 512 pixels of 16 bits. Framebuffers, textures, palettes and the font live here.

### CLUT (Color Look-Up Table)

A palette stored in VRAM that 4 bpp and 8 bpp textures point into: 16 or 256 entries of 16-bit colour.

### SPU (Sound Processing Unit)

The PS1's audio chip, with 512 KB of its own RAM and 24 ADPCM voices.

### PCdrv

A debug-link facility that lets the PS1 read files from the host computer as a virtual drive. Play and Run on hardware use it to serve the exported scene files, so nothing needs burning.

### Unirom

A PS1 boot ROM and shell that accepts programs over the serial port. Run on hardware needs it.

### Room and portal

A Room is a box of static geometry that draws only when it can be seen. A Portal is the opening between two Rooms the camera can see through. See [Rooms and portals](../editor/navigation-rooms.md#rooms-and-portals).

### BVH

A tree of bounding boxes over the scene's triangles. Without Rooms, the engine draws only triangles in boxes the camera sees.
