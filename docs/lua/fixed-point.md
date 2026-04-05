# Fixed-Point Math

This is the most important page if you're writing Lua for SplashEdit. **The PS1 has no floating-point unit**, so all math uses fixed-point integers.

## The Rules

1. **Never write decimal literals.** `0.5`, `1.5`, `3.14` - none of these work.
2. **Plain integer division does NOT give you fractions.** `1/2` equals `0`, not `0.5`. This is integer division.
3. **Use `FixedPoint.new()` for ALL fractional values.** This is the only way to create non-integer numbers.
4. **All API functions accept and return fixed-point numbers.** Positions, rotations, distances - everything.

## How It Works

Numbers in SplashEdit Lua are **20.12 fixed-point integers**:

- 20 bits for the integer part
- 12 bits for the fractional part
- Scale factor: **4096** (4096 = 1.0 internally)
- Range: approximately -524288 to +524287

You don't need to think about the internal representation. Just follow the rules above.

## Creating Fractional Values

The **only** way to get a fractional value is through `FixedPoint.new()`:

```lua
-- Create a FixedPoint value and divide it
local one = FixedPoint.new(1)
local half = one / 2              -- 0.5
local quarter = one / 4           -- 0.25
local threeQuarters = one * 3 / 4 -- 0.75
local step = one / 64             -- ~0.015625
local small = one / 16            -- 0.0625
```

!!! danger "Common Mistake"
    ```lua
    -- WRONG: These all equal 0!
    local half = 1/2        -- Integer division: 0
    local frac = 5/4        -- Integer division: 1
    local tiny = 1/64       -- Integer division: 0
    local bad = 0.5         -- Float literal: doesn't work

    -- CORRECT: Use FixedPoint.new()
    local one = FixedPoint.new(1)
    local half = one / 2    -- Proper fixed-point 0.5
    local frac = one * 5 / 4  -- Proper fixed-point 1.25
    local tiny = one / 64   -- Proper fixed-point ~0.016
    ```

## Whole Numbers Are Fine

Plain integers work as-is for whole numbers:

```lua
local ten = 10          -- This is fine
local zero = 0          -- This is fine
local big = 1000        -- This is fine
```

You only need `FixedPoint.new()` when you need a value with a fractional part.

## Rotation Values

Rotations use **pi-units**: 1024 internal units = pi radians = 180 degrees.

| Angle | Lua Value | How to Write |
|-------|-----------|-------------|
| 0 | `0` | `0` |
| 45 degrees | 0.25 pi | `FixedPoint.new(1) / 4` |
| 90 degrees | 0.5 pi | `FixedPoint.new(1) / 2` |
| 180 degrees | 1 pi | `1` |
| 270 degrees | 1.5 pi | `FixedPoint.new(3) / 2` |
| 360 degrees | 2 pi | `2` |

```lua
local one = FixedPoint.new(1)

-- Rotate 90 degrees
Entity.SetRotationY(self, one / 2)

-- Add 180 degrees to current rotation (1 is a whole number, fine as-is)
local rot = Entity.GetRotationY(self)
Entity.SetRotationY(self, rot + 1)

-- Small rotation
Entity.SetRotationY(self, rot + one / 64)
```

## Math Functions

Math functions work with fixed-point numbers. Remember to use `FixedPoint.new()` for fractional arguments:

```lua
local one = FixedPoint.new(1)

-- PSXMath utilities
local clamped = PSXMath.Clamp(value, 0, 100)
local lerped = PSXMath.Lerp(0, 10, one / 2)   -- Interpolate halfway: result is 5
local sign = PSXMath.Sign(-42)                  -- Result: -1
local abs = PSXMath.Abs(-7)                     -- Result: 7
```

## Vec3 Operations

Vectors are tables with `x`, `y`, `z` fields. Fractional components require FixedPoint:

```lua
local one = FixedPoint.new(1)

local pos = Vec3.new(10, 0, 5)
local dir = Vec3.new(one / 2, 0, one / 2)

local sum = Vec3.add(pos, dir)
local dist = Vec3.distance(pos, Vec3.new(0, 0, 0))
```

## String Concatenation

FixedPoint values work with the `..` string operator:

```lua
local pos = self.position
Debug.Log("Position: " .. pos.x .. ", " .. pos.y .. ", " .. pos.z)
```

## Common Gotchas

!!! tip "onUpdate dt is fixed-point"
    The `dt` parameter passed to `onUpdate(self, dt)` is a 4.12 fixed-point value where 4096 = one 30fps frame. To scale a movement by dt: `local scaled = step * dt / 4096`.

!!! warning "Integer division is silent"
    `1/2` silently evaluates to `0`. There's no error, no warning. Your code runs, but with wrong values. Always use `FixedPoint.new()` for fractions.

!!! warning "Integer overflow"
    Very large multiplications can overflow the 32-bit fixed-point range. Keep intermediate values reasonable.

!!! warning "Reuse your FixedPoint local"
    Create `local one = FixedPoint.new(1)` once at the top of your script or function, then derive all fractions from it. Don't call `FixedPoint.new()` inside hot loops.
