# Play, Hardware and Discs

The title bar has the three ways to run a scene: **Play**, **Run on hardware** and **Export**. The **Build** menu holds the same commands with their shortcuts.

| Command | Shortcut | Runs the scene |
|---------|----------|----------------|
| **Play** | ++f5++ | In pcsx-redux, inside the editor |
| **Run on Hardware** | ++ctrl+f5++ | On a PS1 over a serial link |
| **Export Disc Image...** | ++ctrl+b++ | As a `.bin` + `.cue` disc image |

All three save open scripts first and export every scene in the [build list](../editor/scene-settings.md#build-scenes). With an empty build list they export the open scene alone.

## Play

Press **Play**. The editor exports the scene, boots psxsplash in pcsx-redux, and splits the viewport into **Scene** and **Game**, side by side or stacked depending on the window shape. Drag the bar between them to resize.

Click the Game view to give it the keyboard:

| Keyboard | Controller |
|----------|------------|
| Arrow keys | D-pad |
| ++z++ ++x++ ++a++ ++s++ | Cross, Circle, Square, Triangle |
| ++q++ ++w++ ++1++ ++2++ | L1, R1, L2, R2 |
| ++enter++ | Start |
| ++backspace++ | Select |

A connected gamepad always drives controller 1, focused or not.

**Live edits.** The scene stays editable while the game runs. A moment after you stop editing, the editor exports again and the game reloads the scene in place, so you can move a light or retune a player value without restarting.

**Output.** While the game runs, the status bar shows its state. Click it to see what the game printed, including `Debug.Log` lines, with a **Stop** button. Press **Play** again (it reads **Stop** now) or ++f5++ to end the game.

## Setting up the tools

The archive ships everything Play needs, and the editor finds it on its own. If something is missing, the editor opens **Set up Play** instead of playing:

- **PlayStation executable**: the psxsplash build to boot. Defaults to `engine/psxsplash.ps-exe`.
- **BIOS image**: optional. Left empty, pcsx-redux uses its own BIOS.

**Locate** picks a file. The same window appears as **Set up Export** for the disc build and **Set up Run on hardware** for the serial port.

The editor also reads these environment variables, before falling back to the bundled copies: `SPLASHEDIT_REDUX`, `SPLASHEDIT_PSXSPLASH`, `SPLASHEDIT_BIOS`, `SPLASHEDIT_DISC_ENGINE` and `SPLASHEDIT_PORT`. If neither is set and no bundled copy is found, it looks for `pcsx-redux` on your `PATH`.

## Debug overlays

Two entries in the **Build** menu boot a psxsplash build with on-screen diagnostics:

| Overlay | Shows |
|---------|-------|
| **Performance Overlay** | FPS counter and a heap bar |
| **Room Debug Overlay** | The room wireframe, for checking [rooms and portals](../editor/navigation-rooms.md#rooms-and-portals) |

They apply to Play and Run on hardware. Disc images always get the plain CD-ROM build.

## Run on hardware

Run on hardware uploads psxsplash to a real console and then serves the scene files to it over the same cable, so you can test without burning a disc.

You need:

- A PS1 running **Unirom**, sitting at its shell.
- A serial link: a USB serial adapter wired to the console's serial port, or pcsx-redux's SIO1 server for a dry run.

The first time, **Set up Run on hardware** asks for the **Serial port**: `COM3` on Windows, `/dev/ttyUSB0` on Linux, or `tcp:localhost:6699` for pcsx-redux's SIO1 server. Ports found on your machine are listed under **Found:**. The link runs at 115200 baud.

Press **Run on hardware**. The editor exports the scene, sends `psxsplash.ps-exe` to Unirom, and answers the game's file requests (PCdrv) from the export folder. The status bar shows progress and the game's output, like Play. **Stop Hardware Run** in the Build menu ends the session.

## Export a disc image

Press **Export** (++ctrl+b++) and choose where to save the `.bin`. The editor writes an ISO 9660 image in raw Mode 2 sectors plus its `.cue`, with the CD-ROM build of psxsplash (`engine/psxsplash-cdrom.ps-exe`) and every scene in the build list. The status bar reports the file names and size when it is done.

Disc options are in the scene settings under **Disc** (select nothing to see them):

| Field | Effect |
|-------|--------|
| **Volume label** | The disc's volume name, up to 31 characters. Empty uses the scene name. |
| **Licence** | The licence data in the disc's first 16 sectors. **None** boots in emulators and on modded consoles. **psxsplash** uses the psxsplash licence; the boot logo reads "Created with psxsplash". **Choose file...** takes your own licence file. A retail console's BIOS checks this before booting. |

The image runs in any PS1 emulator, or burn it to a CD-R for a console that boots burned discs.

!!! note "No CD-DA tracks"
    The image holds one data track. `Audio.PlayCDDA` plays audio tracks from a disc, so it has nothing to play on an editor-made image, nor under Play or Run on hardware, which load files over PCdrv. Use [audio clips](../editor/audio.md) for music.
