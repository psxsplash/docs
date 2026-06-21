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
| MIPS Cross-Compiler | Compiles C++ to PS1 machine code | Click "Install" (Win/Linux) |
| GNU Make | Build system | Click "Install" (Win/Linux) |
| PCSX-Redux | PS1 emulator for testing | Click "Download" |
| psxavenc | Audio conversion to ADPCM | Click "Download" |
| mkpsxiso | ISO image creation | Click "Download" |

Each tool shows a green **Ready** badge when installed. On **Windows** and **Linux**, SplashEdit handles downloading and setting up all of these for you. On **macOS**, some tools require manual Terminal commands first — the Dependencies tab shows yellow badges and the exact commands to run (see [macOS Setup](#macos-setup) below).

!!! note "Minimum requirements"
    You need at minimum the **MIPS compiler** and **Make** to build. **PCSX-Redux** is needed for emulator testing and Lua compilation. **mkpsxiso** is only needed for ISO builds.

### macOS Setup

On macOS, some tools can't be auto-installed. The Dependencies tab shows yellow badges for anything that requires manual action — **Needs setup**, **Needs CLT** (Xcode Command Line Tools), **Needs brew** (Homebrew), or **Needs deps** (build dependencies) — with the exact Terminal commands inline. Install the prerequisites below, then click **Refresh** — the badges will turn green.

**Prerequisites (run in Terminal before opening SplashEdit):**

```bash
# Install Xcode Command Line Tools (provides GNU Make)
xcode-select --install

# Install the MIPS cross-compiler helper
brew install nikitabobko/tap/brew-install-path

# Download the compiler formulas from pcsx-redux
curl -LO https://raw.githubusercontent.com/grumpycoders/pcsx-redux/main/tools/macos-mips/mipsel-none-elf-binutils.rb
curl -LO https://raw.githubusercontent.com/grumpycoders/pcsx-redux/main/tools/macos-mips/mipsel-none-elf-gcc.rb

# Install (builds from source — expect 15-30 minutes)
brew install-path ./mipsel-none-elf-binutils.rb
brew install-path ./mipsel-none-elf-gcc.rb
```

!!! tip "Don't have Homebrew?"
    Install it first: `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

!!! info "Why so many steps?"
    There is no prebuilt MIPS compiler for macOS. The formulas above are maintained by the [pcsx-redux](https://github.com/grumpycoders/pcsx-redux) project (the same team behind the PCSX-Redux emulator). The `brew-install-path` helper is needed because Homebrew no longer supports installing from local formula files directly.

**psxavenc build dependencies** — psxavenc has no prebuilt macOS binary, so SplashEdit builds it from source. The Dependencies tab will show **Needs deps** until you install:

```bash
brew install meson pkg-config ffmpeg
```

Once installed, click **Refresh**, then **Download** to build psxavenc.

The remaining tools are handled automatically:

- **PCSX-Redux** — Click "Download." SplashEdit downloads the macOS ARM build, creates a launcher wrapper, and configures font paths.
- **mkpsxiso** — Click "Download." Works the same as Windows/Linux.

## Verifying Installation

When all required tools show green badges, the Control Panel status bar shows **"Ready"**. You're good to go.

If SplashEdit detects a missing toolchain on first launch, it automatically opens the Control Panel to guide you through setup.
