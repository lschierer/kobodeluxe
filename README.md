# Kobo Deluxe

An enhanced port of the classic scrolling arcade shooter XKobo.

> **This is a community fork** maintained at [lschierer/kobodeluxe](https://github.com/lschierer/kobodeluxe).
> It modernises the build system and dependencies so the game runs on current
> operating systems, starting with macOS.  The gameplay is unchanged.

---

## What changed from upstream

- Build system replaced with **CMake 3.15+** (was GNU Autotools)
- Graphics and input updated to **SDL 2** (was SDL 1.2)
- **glSDL** (SDL 1-era OpenGL shim) removed; SDL2 window-surface rendering used instead
- First-run user directory (`~/.kobodeluxe`) created automatically
- In-game music correctly respects the "Enable Music" preference

---

## Building on macOS

### Prerequisites

Install [Homebrew](https://brew.sh), then:

```sh
brew install cmake sdl2 sdl2_image
```

Xcode Command Line Tools are also required (`xcode-select --install`).

### Build

```sh
git clone https://github.com/lschierer/kobodeluxe.git
cd kobodeluxe
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

This produces `build/src/kobodl.app`.  Double-click to launch, or:

```sh
open build/src/kobodl.app
```

### Install (optional)

```sh
cmake --install build
```

Installs to `/usr/local` by default.  Pass `-DCMAKE_INSTALL_PREFIX=<path>` to
the configure step to change this.

---

## Building on Linux

Install SDL2 and SDL2_image via your package manager, e.g.:

```sh
# Debian / Ubuntu
sudo apt install cmake libsdl2-dev libsdl2-image-dev

# Fedora
sudo dnf install cmake SDL2-devel SDL2_image-devel
```

Then follow the same `cmake -B build … cmake --build build` steps above.

---

## Controls

| Action | Keys |
|--------|------|
| Move | Arrow keys, or numpad 2/4/6/8 |
| Move (diagonals) | Numpad 1/3/7/9 |
| Fire | Left Shift, Left Ctrl |
| Menu / Pause | Escape |

Mouse and joystick input can be enabled in the in-game Options menu.

---

## Configuration

All settings can be changed from the in-game menus or by editing
`~/.kobodeluxe/kobodeluxe.cfg` directly.  Common command-line overrides:

| Flag | Effect |
|------|--------|
| `-fullscreen` | Start in fullscreen mode |
| `-nosound` | Disable audio |
| `-nomusic` | Disable in-game music |
| `-width W -height H` | Set window size |
| `-maxfps N` | Cap frame rate (75 is good for laptops) |

Run `kobodl -help` for the full option list.

---

## Description

Kobo Deluxe is a 3rd-person scrolling 2D shooter with a simple and responsive
control system.  Tackle waves of enemy ships that shoot at you, chase you,
circle around you, or launch other ships at you, while you destroy
labyrinth-shaped bases across 50 levels of smoothly escalating difficulty.

A "Classic" mode reproduces the original XKobo gameplay exactly.

---

## Credits

Originally written by **Akira Higuchi**.  Turned into Kobo Deluxe by
**David Olofson**.  SDL2 port and CMake modernisation by contributors to
this fork.

See the [`README`](README) file for the full credits and copyright notice.
