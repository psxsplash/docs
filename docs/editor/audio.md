# Audio

Sound effects and music are **audio clips**: WAV files the editor converts to the PS1's SPU format, **SPU-ADPCM**, at export. All clips share the SPU's **512 KB** of sound RAM; the **SPU RAM** meter in the status bar shows how much a scene uses.

## Adding a clip

**Add Object > Audio clip**, or **Add component > Audio clip** on any object, then pick a WAV file.

| Field | Meaning |
|-------|---------|
| **Clip** | A WAV file. Channels are averaged to mono. |
| **Source** | The WAV as it is, and the sound RAM the exported clip takes. |
| **Name** | What scripts play it by. Starts as the object's name. |
| **Sample rate** | The clip is resampled to this at export. Lower takes less sound RAM and sounds duller. 22050 is a good middle. |
| **Loop** | Plays again from the start when it ends, until stopped. |
| **Trim silence** | Cuts the quiet part at the start, so the sound starts the moment it is triggered. |

## Playing a clip

```lua
Audio.Play("creak")                 -- volume 100, centre
Audio.Play("creak", 128, 0)         -- loudest, hard left
local channel = Audio.Play("music", 90)
Audio.Stop(channel)
```

`Audio.Play` returns the channel (0 to 23), or -1 if all 24 voices are busy. Volume runs 0 to 128, pan 0 (left) to 127 (right), 64 centre. In the [block editor](block-editor.md), **play sound** takes a clip name from a list of the scene's clips. A [cutscene](timeline.md#audio-and-skin-events-cutscenes) can play clips at set frames.

See the [`Audio` API](../lua/api-reference.md#audio).

## CD-DA

`Audio.PlayCDDA` plays audio tracks burned on a disc. The editor's disc images hold one data track, and Play and Run on hardware load files over PCdrv, so these calls have nothing to play. A scene with [world streaming](world-streaming.md) on ignores them as well. Use audio clips for music.

## Keeping it small

- ADPCM takes about 0.57 bytes per sample, so a 5-second clip at 22050 Hz is about 63 KB and at 11025 Hz about 31 KB. Drop to 11025 Hz for effects.
- Keep clips short; loop a short bar instead of a long track.
- There are 24 voices. Plan for the worst case of sounds at once.
