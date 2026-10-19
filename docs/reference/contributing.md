# Contributing

psxsplash is open source and pull requests are welcome.

!!! note "Share what you make"
    If you build something with psxsplash, show it on the [PSX.Dev](https://psx.dev) or Bandwidth Discord servers.

## Repositories

| Repository | Contents |
|-----------|----------|
| [Editor and downloads](https://github.com/psxsplash/splashedit-ng) | The editor, its exporter and the packaged downloads |
| [Engine](https://github.com/psxsplash/psxsplash) | The C++ PS1 engine (built on PSYQo) |
| [Docs](https://github.com/psxsplash/docs) | These pages |

Report a problem from **Help > Report a Problem** in the editor, or open an issue on the editor's repository.

## Ways to help

- Bug reports with a project that reproduces them.
- Fixes to anything in [Known Issues](known-issues.md).
- Editor controls for engine features that have none yet: agents, sprites, networking, and the runtime-light switch.
- Corrections and examples for these docs.

## Building the editor

It needs CMake 3.24 or newer, a C++20 compiler, SDL3 and FreeType. Dear ImGui and stb are fetched at configure time.

```bash
cmake -S . -B build -G Ninja
cmake --build build
./build/splashedit
```

## Working on these docs

The docs are MkDocs Material with [mike](https://github.com/jimporter/mike) for versions.

```bash
pip install -r requirements.txt
mkdocs serve              # live preview
mkdocs build --strict     # what CI checks
```

| Branch or tag | Published as |
|---------------|--------------|
| `main` | `nightly`, also aliased `latest`: the docs for the current nightly editor |
| tag `v*` | A numbered version, such as `2.4.0`, also aliased `latest` |

The version menu in the header lists them all. `unity` is the legacy documentation for the Unity editor (2.4).
