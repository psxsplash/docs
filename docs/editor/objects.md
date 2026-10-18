# Objects & Components

Everything in a scene is an object: a name, a transform, a **Start active** switch, and any number of components. An object with no components is an empty, useful for grouping.

## Adding objects

**+** in the Scene panel or **Object > Add Object...** (++ctrl+a++) opens a searchable list of starting points:

| Object | Starts with |
|--------|-------------|
| Empty | Only a transform |
| Mesh | A Mesh component; pick its file in the inspector |
| Point light | A Light |
| Player | A Player |
| Trigger box | A Trigger box |
| Interactable | An Interactable |
| Audio clip | An Audio clip |
| Script | A Script |
| Navigation | A Navigation component |
| Room | A Room |
| Portal | A Portal |
| Agent (NPC) | An Agent |

New objects appear in front of the camera, after the selected object. Dropping a model file on the viewport or the scene tree also makes an object; see [Importing](importing.md).

**Add component** at the bottom of the inspector (or **Object > Add Component...**) adds a component to the selected object. Each component type can be added once per object. Collider and Skinned animation need a Mesh first.

## Transform

**Position** in metres and **Rotation** in degrees, both relative to the parent, and **Scale** per axis. Drag the viewport gizmo or type values. At export, positions are divided by the scene's **GTE scaling** (see [Scene Settings](scene-settings.md#rendering)) into PS1 units, so everything has to sit within 8 x the scaling of the origin: 800 m at the default of 100.

**Start active** off ships the object switched off. Its meshes are not drawn until a script calls `Entity.SetActive(object, true)`; its lights still bake.

## Components

### Mesh

Draws a model. **Model** is the `.mesh` file (or pick a `.glb`, `.gltf`, `.fbx`, `.obj` or `.blend` to import one); then one **Texture** and colour per material, **Lighting** and **Smooth normals**. See [Materials & Bit Depth](materials.md) and [Lighting](lighting.md).

### Collider

| Shape | Effect |
|-------|--------|
| **Static** | Solid scenery. Feeds the navigation bake: the player walks on static floors and is stopped by static walls. |
| **Dynamic** | Can be moved by scripts. Tested against the player every frame, and the only kind that fires `onCollideWithPlayer`. |
| **None** | No collision. |

!!! note "Rotating a dynamic collider"
    Its box does not rotate with the object. See [Known Issues](../reference/known-issues.md).

### Script

Runs a script for this object. **File** points at a `.lua` or `.blocks` file; **New blocks** and **New Lua** create one in `scripts/` named after the object, and **Edit script** opens it. Each object gets its own copy of the script's state. See [Lua scripting](../lua/index.md) and [The Block Editor](block-editor.md).

### Light

Lights the meshes around it. See [Lighting](lighting.md).

### Player

Where the player starts and how it moves. One per scene.

| Field | Meaning |
|-------|---------|
| **Height** | Eye height |
| **Radius** | How wide the player is, for walls and the nav bake |
| **Move speed** | Walking speed |
| **Sprint speed** | Speed while the sprint button is held |
| **Jump height** | How high a jump goes |
| **Gravity** | How fast the player falls; about 9.8 is Earth |
| **Step height**, **Max slope** | What the nav bake treats as walkable |
| **Cell size**, **Cell height** | The nav bake's voxel size |

See [Player, Navigation & Rooms](navigation-rooms.md).

### Navigation

Bakes walkable ground without a player, or overrides the player's bake settings: **Agent height**, **Agent radius**, **Step height**, **Max slope**, **Cell size**, **Cell height**.

### Trigger box

An axis-aligned box (**Size**, in metres, centred on the object) that calls a script when the player walks in or out. **Script** is the trigger's own script, separate from the object's Script component; it gets `onTriggerEnter` and `onTriggerExit`. **New blocks** and **New Lua** make a `<name>_trigger` file.

### Interactable

Calls the object's script (`onInteract`) when the player presses a button nearby.

| Field | Meaning |
|-------|---------|
| **Radius** | How close the player has to be |
| **Button** | The pad button that uses it (Cross by default) |
| **Repeatable** | On: usable again after **Cooldown**. Off: works once. |
| **Show prompt**, **Prompt** | Shows a UI canvas while the player is in range. The canvas name must fit in 15 characters. |
| **Require facing**, **Facing angle** | The player has to look toward the object |

### Audio clip

A sound scripts can play by name. See [Audio](audio.md).

### Skinned animation

Plays animation clips on a skinned mesh. See [Skinned Animation](skinned-animation.md).

### Agent (NPC)

A character the engine walks along the nav regions: patrol waypoints, vision and hearing, and a skin clip per state. See [Agents](agents.md).

### Room and Portal

Room boxes and the openings between them, for culling indoor scenes. See [Player, Navigation & Rooms](navigation-rooms.md#rooms-and-portals).

## Copy, paste and duplicate

**Duplicate** (++ctrl+d++), **Cut**, **Copy** and **Paste** work on the selected object and its children. Paste puts the copy after the selection. Names stay unique: a second "Crate" becomes "Crate (2)".

## Names matter

Scripts, timeline tracks, portals and interact prompts find objects by name. Rename with ++f2++ or by double-clicking the name in the inspector. Keep names unique among objects a script looks up.
