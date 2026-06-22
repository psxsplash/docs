# Installation

## Prerequisites

- **Unity 6000.0+** with a **Universal Render Pipeline (URP)** project
- **Git** installed and available in your system PATH
- **Windows**, **Linux**, or **macOS** (Apple Silicon)

!!! important "Universal Render Pipeline"
    Your Unity project must use the Universal Render Pipeline. When creating a new project, select the **Universal 3D** template.

## Step 1: Install SplashEdit in Unity

1. Download the latest SplashEdit release as a `.tgz` file from [the releases page](https://github.com/psxsplash/splashedit/releases)
2. In Unity, go to **Window -> Package Manager**
3. Click the **+** button and select **Add package from tarball...**
4. Select the downloaded `.tgz` file
5. Unity imports the package

## Step 2: Open the Control Panel

Go to **PlayStation 1 -> SplashEdit Control Panel** (or press ++ctrl+shift+l++).

The Control Panel is your central hub for dependencies, scene management, and building.

## Step 3: Install the Native Project

In the Control Panel's **Dependencies** tab:

1. Under **Native Project**, click the release dropdown and select the version that **matches your SplashEdit package version**
2. Click **Clone** to download the [psxsplash](https://github.com/psxsplash/psxsplash) C++ runtime
3. Wait for the clone to complete (this uses Git under the hood)

!!! tip
    Alternatively, click **Browse** to point to a local copy of the psxsplash source if you already have it cloned.

## Step 4: Install the Toolchain

Still in the **Dependencies** tab, the toolchain section shows the status of each required tool:

| Tool | Purpose | Install Method |
|------|---------|---------------|
| MIPS Cross-Compiler | Compiles C++ to PS1 machine code | Click "Install" |
| GNU Make | Build system | Click "Install" |
| PCSX-Redux | PS1 emulator for testing | Click "Download" |
| psxavenc | Audio conversion to ADPCM | Click "Download" |
| mkpsxiso | ISO image creation | Click "Download" |

Each tool shows a green **Ready** badge when installed. SplashEdit handles downloading and setting up all of these for you.

!!! note "Minimum requirements"
    You need at minimum the **MIPS compiler** and **Make** to build. **PCSX-Redux** is needed for emulator testing and Lua compilation. **mkpsxiso** is only needed for ISO builds.

### Platform notes

On **Windows** and **Linux**, the MIPS cross-compiler and Make are installed automatically when you click **Install**.

!!! info "macOS (Apple Silicon)"
    There is no prebuilt MIPS compiler for macOS, so the setup wizard **guides you through a one-time manual install** instead of installing silently:

    1. **Xcode Command Line Tools** — required for the compiler. The wizard prompts you if they're missing.
    2. **Homebrew** — the macOS package manager. If it's not installed, the wizard copies the official install command to your clipboard.
    3. **MIPS cross-compiler** — installed through Homebrew using the grumpycoders formulae:
        ```bash
        brew install nikitabobko/tap/brew-install-path
        curl -LO https://raw.githubusercontent.com/grumpycoders/pcsx-redux/main/tools/macos-mips/mipsel-none-elf-binutils.rb
        curl -LO https://raw.githubusercontent.com/grumpycoders/pcsx-redux/main/tools/macos-mips/mipsel-none-elf-gcc.rb
        brew install-path ./mipsel-none-elf-binutils.rb
        brew install-path ./mipsel-none-elf-gcc.rb
        ```
       The wizard copies these commands to your clipboard for you.

    **PCSX-Redux** is downloaded automatically as a `.dmg`, unpacked, and wrapped so SplashEdit can launch it (bundled fonts are copied to `~/Library/Fonts/` and the download is de-quarantined). When checking for tools, SplashEdit also searches the Homebrew paths (`/opt/homebrew/bin`, `/usr/local/bin`) and accepts the `mipsel-none-elf` compiler variant in addition to `mipsel-linux-gnu`.

## Verifying Installation

When all required tools show green badges, the Control Panel status bar shows **"Ready"**. You're good to go.

If SplashEdit detects a missing toolchain on first launch, it automatically opens the Control Panel to guide you through setup.
