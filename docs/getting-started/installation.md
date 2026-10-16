# Installation

psxsplash ships as one archive per OS from the [nightly release](https://github.com/psxsplash/splashedit-ng/releases/tag/nightly), rebuilt from every change to the editor. Each archive holds:

| Folder or file | What it is |
|----------------|------------|
| `splashedit` (`splashedit.exe` on Windows, `SplashEdit.app` on macOS) | The editor |
| `engine/` | psxsplash builds: `psxsplash.ps-exe` for Play and Run on hardware, `psxsplash-cdrom.ps-exe` for disc images, and builds with debug overlays |
| `redux/` | pcsx-redux, the emulator Play boots |
| `examples/courtyard/` | The example project |
| `assets/` | Fonts and icons the editor uses |

Unpack it anywhere and start the editor. Nothing else needs installing. On macOS the other folders sit inside `SplashEdit.app`.

## Windows

Windows 10 or later. Download `splashedit-windows.zip`, extract it, and run `splashedit.exe`.

The executable is not signed, so SmartScreen may stop the first launch. Click **More info**, then **Run anyway**.

## Linux

x86-64 with glibc 2.38 or newer: Ubuntu 24.04, Debian 13, Fedora 39 and later.

```bash
tar xzf splashedit-linux.tar.gz
cd splashedit
./splashedit
```

`./install-desktop-entry.sh` adds the editor to your applications menu. Run it again if you move the folder.

## macOS

Apple silicon only. Download `splashedit-macos.tar.gz` and unpack it. The app is not notarized, so clear the download quarantine once before the first launch, then open `SplashEdit.app`:

```bash
tar xzf splashedit-macos.tar.gz
xattr -dr com.apple.quarantine splashedit
```

## Optional extras

- **Blender**, to import `.blend` files directly. The editor runs Blender in the background to read them. It looks for `blender` on your `PATH`, then the usual install folders (`C:/Program Files/Blender Foundation/` on Windows, `/Applications/Blender.app` on macOS). Set `SPLASHEDIT_BLENDER` to the executable to pick one yourself. Every other model format works without it.
- **A BIOS image.** pcsx-redux boots with its own BIOS when none is set. To use a real one, pick it under **BIOS** in the Play setup window (see [Play, Hardware and Discs](play-and-export.md#setting-up-the-tools)).
- **A serial cable and Unirom** on a real console, for [Run on hardware](play-and-export.md#run-on-hardware).

## Updating

Download the archive again and unpack it over the old folder, or next to it. Projects live in their own folders, so nothing in a project changes when you update the editor.
