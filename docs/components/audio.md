# Audio

SplashEdit converts Unity AudioClips to PS1 SPU ADPCM format for playback on real hardware.

## Adding Audio

1. Add a `PSXAudioClip` component to any GameObject in your scene
2. Assign a Unity AudioClip
3. Configure settings

You can put multiple `PSXAudioClip` components on the same GameObject or spread them across different objects - placement doesn't matter, they're all collected at export time.

## PSXAudioClip Settings

| Field | Default | Description |
|-------|---------|-------------|
| Clip Name | (required) | Unique identifier used in Lua. `Audio.Play("clip_name")` |
| Clip | (required) | Unity AudioClip source |
| Sample Rate | 22050 | Target sample rate in Hz (8000-44100). Lower = smaller, worse quality. |
| Loop | false | Whether the clip loops when played |
| Default Volume | 100 | Default playback volume (0-127) |

## Playing Audio from Lua

```lua
-- Play by name with volume and pan
Audio.Play("collect", 127, 64)      -- full volume, centered
Audio.Play("music", 80, 0)          -- 80 volume, left channel
Audio.Play("ambience", 60, 127)     -- 60 volume, right channel

-- Audio.Play returns a channel number (0-23) or -1 if all voices are busy
local ch = Audio.Play("music", 100, 64)

-- Find clip index by name
local idx = Audio.Find("collect")

-- Stop a specific channel
Audio.Stop(ch)

-- Adjust volume and pan mid-playback
Audio.SetVolume(ch, 50, 64)

-- Stop everything
Audio.StopAll()
```

## PS1 Audio Constraints

| Resource | Limit |
|----------|-------|
| SPU RAM | 512KB total for all audio samples |
| Hardware voices | 24 simultaneous channels |
| Format | ADPCM (lossy compression) |

!!! warning "Voice limit"
    `Audio.Play` returns **-1** if all 24 hardware voices are busy. Check the return value if you need to know whether playback started.

## Volume and Pan

| Parameter | Range | Description |
|-----------|-------|-------------|
| Volume | 0-128 | 0 = silent, 128 = maximum |
| Pan | 0-127 | 0 = full left, 64 = center, 127 = full right |

Default values if not specified: volume 100, pan 64 (center).

## Tips

!!! tip "Sample rate"
    **11025 Hz** is fine for short sound effects (clicks, hits, pickups). **22050 Hz** is good for voice and music. **44100 Hz** uses double the SPU RAM of 22050 Hz - only use it if quality really matters.

!!! tip "Keep clips short"
    SPU RAM is only 512KB. A 5-second clip at 22050 Hz uses roughly 20KB in ADPCM. A 30-second music track uses roughly 120KB. Budget carefully.

!!! tip "Audio in cutscenes"
    [Cutscenes](cutscenes.md) have built-in audio events that trigger at specific frames. Use those for synchronized audio instead of playing clips from Lua callbacks.
