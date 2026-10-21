# Shadows

A **blob shadow** is a soft dark disc on the ground under an object, the shadow most PS1 games used to show where a character is and how high it jumps.

## Turning one on

Select an object with a mesh. In the **Mesh** section, set **Shadow** to **Blob**.

| Field | Meaning |
|-------|---------|
| **Shadow** | **Off** or **Blob**. |
| **Size** | Radius of the blob when the object is on the ground, in metres. Switching to Blob sets it from the size of the object; type a value to change it. |
| **Darkness** | How much the blob darkens the ground at its centre, from 0 (no change) to 100 (black). It fades out toward the rim. |

The Scene view shows each blob under the object, at the size you set.

## On the console

- The blob lies on the [navigation floor](navigation-rooms.md) under the object and tilts with it on ramps.
- An object that starts off the nav floor and does not move gets a flat blob under the bottom of its bounds. An object that has moved off the nav floor has no blob until it is over the nav floor again.
- The blob shrinks and fades as the object rises, down to zero at 2.5 m above the ground.
- It fades out near walk-off edges and in [fog](scene-settings.md).
- The console subtracts the blob's colour from the ground, so blobs show most on bright ground.
- The blob is an eight-sided fan. Very large blobs show their corners.
- At most **32** blobs per scene. Export lists the extra ones in a warning and skips them.

Each blob costs a little frame time. With no blobs in a scene the frame time is the same as without the feature.

## Warnings

- **"its Blob shadow has no navigation floor under it where the scene starts"**: the object starts off the nav floor, so its blob is flat under the object until the object moves over the nav floor. Move the object, or keep it for a prop that stands in a gap of the nav floor.
- **"the scene has more blob shadows than the PS1 draws"**: set some to Off.
