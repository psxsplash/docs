# Textures & VRAM

The PS1 has a fixed 1MB of VRAM (1024x512 pixels at 16-bit). SplashEdit manages VRAM allocation automatically, but understanding the layout helps you optimize.

!!! danger "Texture Size Requirements"
    All textures **must be power-of-two in both width and height**. The maximum texture size is **256x256**. SplashEdit does NOT validate this for you. Using non-power-of-two or oversized textures will produce broken results.

    Valid sizes: 16x16, 32x32, 64x64, 128x128, 256x256 (and non-square like 32x64, 64x128, etc.)

## VRAM Layout

The PS1 VRAM is 1024x512 pixels. Framebuffers are arranged vertically (stacked), not side-by-side:

```
  0         320       640       960  1024
  +---------+---------+---------+----+
  |         |                   |Font|
  | Frame-  |                   | Col|
  | buffer  |  Texture Atlas    | (64|
  | 1       |  Space            | px)|
  | 320x240 |                   |    |
  |---------|                   |    |  240
  |         |                   |    |
  | Frame-  |                   |    |
  | buffer  |                   |    |
  | 2       |                   |    |
  | 320x240 |                   |    |
  |         |                   |    |  480
  +---------+-------------------+----+  512
```

- **Framebuffers**: Two 320x240 buffers stacked vertically for double-buffering (mandatory, uses 320x480 of VRAM)
- **Font Column**: x=960-1023, reserved for system font and up to 3 [custom fonts](fonts.md)
- **Texture Atlas Space**: Everything else is available for your textures and CLUTs (color palettes)

## Bit Depth

Each object's texture can use one of three bit depths:

| Bit Depth | Colors | VRAM Width Multiplier | Best For |
|-----------|--------|----------------------|----------|
| 4-bit | 16 colors | 1/4x | Simple textures, UI, fonts |
| 8-bit | 256 colors | 1/2x | Most game textures |
| 16-bit | 32768 colors | 1x | Photos, complex gradients |

A 64x64 texture at 4-bit only uses 16x64 pixels of VRAM space. The same texture at 16-bit uses 64x64. Lower bit depth means **more textures fit in VRAM**.

## Texture Quantization

SplashEdit automatically quantizes your Unity textures to PS1-compatible formats:

1. **K-Means color clustering** finds the optimal palette for the target color count
2. **KD-Tree nearest neighbor** maps each pixel to the closest palette entry
3. **Floyd-Steinberg dithering** distributes quantization error to neighboring pixels for smoother gradients

The result is a palette-indexed texture (4-bit or 8-bit) or direct-color (16-bit) packed into VRAM.

## Color Palettes (CLUTs)

4-bit and 8-bit textures each have a **CLUT** (Color Look-Up Table) stored separately in VRAM. CLUTs are small (16 or 256 entries at 16 bits each) and are packed into available VRAM space.

## Deduplication

If multiple objects share the **same source texture and bit depth**, SplashEdit stores it only once. The deduplication check compares the Unity texture asset reference and the bit depth setting. This happens automatically during export.

## Tips

!!! tip "Use 4-bit aggressively"
    Many PS1 games used 4-bit textures everywhere. 16 colors with dithering looks surprisingly good on a CRT or low-resolution display. Saves 4x VRAM vs 16-bit.

!!! tip "Keep textures small"
    Common PS1 texture sizes: **32x32**, **64x64**, **128x128**. Larger textures eat VRAM fast. Remember: max 256x256, power-of-two only.

!!! tip "Share materials"
    Every unique material creates a separate texture. Two objects with identical materials share one texture. Two objects with different materials pointing to the same image create two separate textures.
