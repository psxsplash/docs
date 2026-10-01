# Glossary

Terms used throughout these docs that assume PS1-specific or SplashEdit-specific
background.

### OT (Ordering Table)

A fixed-size array of buckets used to depth-sort GPU primitives before they're
sent to the GPU. Each primitive is linked into the bucket matching its depth; the
PS1 has no Z-buffer, so this is how the engine approximates depth sorting. See
[OT Size and Bump Size Tuning](memory.md#ot-size-and-bump-size-tuning).

### GTE (Geometry Transformation Engine)

The PS1 CPU's coprocessor 2, a fixed-function unit that does matrix/vector math -
transforms, projection, lighting - in hardware instead of on the R3000A's integer
core. SplashEdit's exported coordinates and rotations are scaled to suit GTE
fixed-point input (see [GTE Scaling](../components/player.md#gte-scaling)).

### SIO0

The PlayStation's first serial port, which drives controllers and memory cards.
Distinct from SIO1, the link-cable port.

### SIO1

The PlayStation's second serial port - the link-cable / "serial port" port, used
for [Networking](../components/networking.md). Distinct from SIO0.

### splashpack

The binary format SplashEdit exports a scene into. The psxsplash runtime reads a
`.splashpack` file to load a scene's objects, textures, nav data, UI, and
everything else authored in Unity.

### Bump allocator

A per-frame, reset-every-frame allocator used for the GPU command buffer. "Bump"
because allocating just advances (bumps) a pointer; nothing is freed
individually, the whole thing resets at the start of the next frame. See
[OT Size and Bump Size Tuning](memory.md#ot-size-and-bump-size-tuning).

### VRAM

The PS1 GPU's dedicated video memory: 1MB, addressed as a 1024x512 pixel grid.
Framebuffers, textures, CLUTs, and fonts all live here. See
[Textures & VRAM](../components/textures.md).

### CLUT (Color Look-Up Table)

A small palette stored in VRAM that 4-bit and 8-bit indexed textures point into.
A 4-bit texture's CLUT has 16 entries, an 8-bit texture's has 256, each entry a
16-bit color.

### SPU (Sound Processing Unit)

The PS1's audio chip, with 512KB of its own dedicated RAM (separate from main RAM
and VRAM) and 24 simultaneous ADPCM voices.

### PCdrv

A BIOS/emulator facility that exposes the host filesystem to the PS1 as a virtual
drive over the debug link. SplashEdit's emulator and real-hardware builds use it
to stream scene files instead of baking everything onto a disc image during
development. See [Build Targets](../getting-started/building.md#build-targets).
