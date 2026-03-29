# Custom Fonts

SplashEdit supports up to **3 custom fonts per scene** (limited by VRAM space in the font column).

## Creating a Font

1. In the Project window, right-click and select **Create -> PSXSplash -> Font Asset**
2. This creates a `PSXFontAsset` ScriptableObject

## Font Configuration

You have two options:

### Option A: From a TrueType/OTF Font

1. Assign a Unity **Font** asset to the **Source Font** field
2. Set the **Font Size** (6-32 pixels)
3. Click **Generate Bitmap** to rasterize it into a glyph grid

This auto-generates a bitmap texture and calculates per-character advance widths.

### Option B: From a Bitmap

1. Create a **256-pixel-wide** texture with your glyph grid
2. Assign it to the **Font Texture** field
3. Set **Glyph Width** and **Glyph Height** (must divide evenly into 256, so valid sizes are: 4, 8, 16, 32)

The grid should contain ASCII characters 0x20-0x7F (96 glyphs total) laid out left-to-right, top-to-bottom.

## PSXFontAsset Fields

| Field | Description |
|-------|-------------|
| Source Font | Unity Font asset (for TrueType generation) |
| Font Size | Pixel size (6-32) |
| Font Texture | 256px-wide bitmap glyph grid |
| Glyph Width | Cell width in pixels |
| Glyph Height | Cell height in pixels |

## Using Custom Fonts

Fonts can be assigned at two levels:

1. **Canvas default** - Set as the **Default Font** on a `PSXCanvas`. All text elements in that canvas use this font unless overridden.
2. **Element override** - Set as **Font Override** on individual `PSXUIText` elements.

Priority: Element Override -> Canvas Default -> System Font

## VRAM Placement

Custom fonts are stored in the **font column** (x=960-1023, 64 pixels wide):

| Slot | Y Position | Max Height | Notes |
|------|-----------|------------|-------|
| Font 1 | 0 | 256 pixels | Most space |
| Font 2 | 256 | 208 pixels | |
| System Font | 464 | 48 pixels | Always present, built into PSYQo |

!!! note
    The system font is always available even without custom fonts. It's a basic monospace font rendered by PSYQo's `chainprintf` system.

## Known Issues

!!! bug "Character cropping"
    Custom fonts can have cropping issues on individual characters. Some glyphs may appear slightly cut off. This is a known issue.

## Tips

!!! tip "4-bit quantization"
    Custom fonts are exported as 4-bit (16 color) textures. Simple, high-contrast fonts work best.

!!! tip "Glyph size"
    Smaller glyphs mean more text fits on screen. 8x8 is compact, 16x16 is readable, 32x32 is large. Choose based on how much text you need to display.
