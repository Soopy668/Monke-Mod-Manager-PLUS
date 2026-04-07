# Monke Mod Manager PLUS

A desktop mod manager for **Gorilla Tag (PC)** built with C++ and Qt 6. Browse, install, and manage BepInEx mods without touching the command line.

---

## Features

- **Curated Mods tab** — a hand-picked list of popular mods fetched live from GitHub. Select a mod and click Install; all dependencies are resolved and downloaded automatically in the correct order.
- **Browse Mods tab** — search all Gorilla Tag mods on GitHub by topic, view star counts and version info, and install directly.
- **Utilities tab** — backup/restore your plugins folder and cosmetics, open important game folders, and jump to community links.
- **Automatic game detection** — finds your Gorilla Tag install on launch.
- **Enable / disable mods** — toggle mods on and off without uninstalling them (`.dll` ↔ `.dll.disabled`).
- **Dependency resolution** — curated installs walk the full dependency graph (e.g. installing Monke Map Loader will also install Utilla, Computer Interface, Bepinject, Extenject, and Newtonsoft.JSON automatically).
- Frameless custom title bar with minimize / maximize / close.
- Cross-platform: **Linux**, **Windows**, and **macOS**.

---

## Curated Mod List

| Mod | Author | Group |
|-----|--------|-------|
| BepInEx | BepInEx Team | Core |
| Gorilla Cosmetics | Bobbie | Cosmetic |
| Monke Map Loader | Vadix | Gameplay |
| Space Monke | Bobbie | Gameplay |
| Utilla | Bobbie | Libraries |
| Computer Interface | Toni Macaroni | Libraries |
| Bepinject | Auros | Libraries |
| Extenject | svermeulen | Libraries |
| Newtonsoft.JSON | James Newton-King | Libraries |

The list lives in `resources/mods.json` and is compiled into the binary. To add or update mods, edit that file and rebuild.

---

## Requirements

- **CMake** 3.21 or newer
- **C++20** compiler — GCC 11+, Clang 14+, or MSVC 2019+
- **Qt 6.4+** with the Widgets and Network modules

---

## Building

### Linux (Ubuntu / Fedora)

```bash
# Ubuntu
sudo apt install cmake build-essential qt6-base-dev

# Fedora
sudo dnf install cmake gcc-c++ qt6-qtbase-devel

# Build
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)

# Run
./build/monke-mod-manager-plus
```

### Bazzite / Fedora Atomic (immutable OS)

Use Distrobox to avoid modifying the base OS:

```bash
distrobox create -n dev -i registry.fedoraproject.org/fedora-toolbox:41
distrobox enter dev
sudo dnf install cmake ninja-build gcc-c++ qt6-qtbase-devel git

cd "/home/youruser/path/to/Monke Mod Manager PLUS"
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
./build/monke-mod-manager-plus
```

Alternatively, layer packages on the host (`rpm-ostree install ...` then reboot), or build as a Flatpak (see below).

### Windows

1. Install Qt 6 (MSVC kit) and Visual Studio Build Tools.
2. Open an **x64 Native Tools Command Prompt**.
3. Run:

```bat
cmake -S . -B build -G "Ninja" ^
  -DCMAKE_BUILD_TYPE=Release ^
  -DCMAKE_PREFIX_PATH=C:\Qt\6.7.0\msvc2019_64
cmake --build build
build\monke-mod-manager-plus.exe
```

### macOS

```bash
brew install cmake qt@6 ninja
cmake -S . -B build -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_PREFIX_PATH="$(brew --prefix qt@6)"
cmake --build build
open build/monke-mod-manager-plus.app
```

---

## Packaging

### System-wide install (Linux)
```bash
cmake --install build --prefix /usr
```

### DEB / RPM (CPack)
```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr
cmake --build build -j$(nproc)
cd build && cpack -G DEB && cpack -G RPM
```

### AppImage
```bash
./packaging/appimage/build-appimage.sh
```
Requires: `ninja`, `curl`, Qt 6 dev libs, and `linuxdeploy` (the script downloads it).

### Flatpak
```bash
flatpak-builder --force-clean --user --install-deps-from=flathub \
  --repo=repo builddir packaging/flatpak/com.monke.ModManagerPlus.yaml
flatpak build-bundle repo monke-mod-manager-plus.flatpak com.monke.ModManagerPlus
```

---

## Project Structure

```
Monke Mod Manager PLUS/
├── src/
│   ├── main.cpp              # Entry point
│   ├── MainWindow.cpp/h      # Main application window
│   ├── ModBrowser.cpp/h      # GitHub mod search browser
│   ├── CuratedBrowser.cpp/h  # Curated mod list + dependency installer
│   └── Paths.cpp/h           # Game path discovery helpers
├── resources/
│   ├── mods.json             # Curated mod definitions
│   ├── resources.qrc         # Qt resource bundle
│   └── icons/
│       └── appicon.png
├── packaging/
│   ├── flatpak/
│   ├── appimage/
│   └── com.monke.ModManagerPlus.desktop
├── CMakeLists.txt
└── COMPILE.txt               # Full compile reference
```

---

## Adding Mods to the Curated List

Open `resources/mods.json` and add an entry to the `"mods"` array:

```json
{
  "name"        : "My Mod",
  "tag"         : "",
  "gitPath"     : "GitHubUser/RepoName",
  "releaseId"   : 0,
  "author"      : "Author Name",
  "group"       : "Gameplay",
  "dependencies": ["Utilla"]
}
```

- `releaseId` — which release to use (0 = latest, 1 = second latest, etc.)
- `installPath` — optional, overrides the default install location relative to the game root (e.g. `"BepInEx/plugins"`)
- `dependencies` — list of other mod names from the same file; they will be installed first automatically

Rebuild after editing the file for changes to take effect.

---

## License

See [LICENSE](LICENSE).
