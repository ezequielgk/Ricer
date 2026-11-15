# Ricer

A CLI script to automate applying color schemes (palettes) to wallpapers using [lutgen](https://github.com/ozwaldorf/lutgen-rs).
The script automatically syncs your custom palettes from a repository, processes them, and applies them to your images.

Also included: **Noctalia Ricer**, a sibling script that generates complete [Noctalia](https://github.com/LierB/noctalia) palettes from any wallpaper using [recol](https://github.com/nlkli/recol) — see the [Noctalia Ricer section](#noctalia-ricer-recol-edition) below.

## Dependencies

This script requires the following packages installed on your system:

* **lutgen**: https://github.com/ozwaldorf/lutgen-rs
* **jq**: JSON processor.
* **curl**: For downloading the schemes.
* **tar**: For extracting the downloaded schemes.
* **recol**: For extracting the images schemes.


## Installation

1. Make sure the script has execute permissions:

```bash
chmod +x ricer
```

2. Move the script to your local binaries folder:

```bash
mkdir -p ~/.local/bin
mv ricer ~/.local/bin/ricer
```

3. Make sure `~/.local/bin` is in your `$PATH`. If it isn't, add this to your shell configuration (e.g. `config.fish` or `.bashrc`):

```bash
fish_add_path ~/.local/bin
```

## Usage

Simply run the command in your terminal:

```bash
ricer
```

### How it works:

1. **Sync**: On execution, the script downloads the palette JSON files from the configured repository, processes them, and generates the necessary configuration files in `~/.config/lutgen/`.
2. **Selection**: It will show you an interactive menu to:
   * Choose the folder containing your original wallpapers.
   * Choose between using custom palettes (synced from the repo) or lutgen's built-in palettes.
3. **Processing**:
   * Creates an organized folder structure (`PaletteName/OriginalFolder`).
   * Applies the filter to all images (`jpg`, `jpeg`, `png`, `webp`) within the folder.
   * Processed images are saved automatically, keeping your originals intact.

## Customization

If you want to use your own palette repository, edit the `REPO_URL` variable inside the script:

```bash
REPO_URL="https://github.com/your-user/your-repo"
```

---

# Noctalia Ricer (recol edition)

Generates complete [Noctalia](https://github.com/LierB/noctalia) palettes (Material Design 3 colors + terminal colors) from any wallpaper using [recol](https://github.com/nlkli/recol). Same fzf-driven flow as the ricer: live preview, instant color tuning with re-preview, parallel processing and caching.

### Dependencies

* **recol**: `cargo install --git https://github.com/nlkli/recol --branch main`
* **fzf**: interactive menus and previews
* **chafa**: image previews in the terminal (sixel)
* **ffmpeg**: JPEG fallbacks and image decoding
* **jq** and **python3**: palette generation and validation

### Installation

```bash
cp noctalia-ricer wallpaper-to-noctalia ~/.local/bin/
chmod +x ~/.local/bin/noctalia-ricer ~/.local/bin/wallpaper-to-noctalia
```

### Usage

```bash
noctalia-ricer              # interactive menu (fzf)
wallpaper-to-noctalia IMG   # direct, no menus
```

Flow: pick a full folder or individual images with fzf (multi-select with TAB and sixel previews) → tune saturation, contrast, brightness, temperature, tint, vibrance, hue and normalize (with live preview and a custom palette name option) → Dark / Light / Both → palettes are written to `~/.config/noctalia/palettes/`, where Noctalia picks them up automatically.

### Live palette mode

Menu option **`5. Live palette`** keeps a palette permanently in sync with your wallpaper:

1. It installs a `wallpaper_changed` hook (native Noctalia v5+ hooks) in `~/.config/noctalia/config.toml`, pointing to `~/.config/noctalia/hooks/live-palette.sh`
2. Every time you change the wallpaper, the hook regenerates the palette from the new image using your saved settings — and instantly re-applies it via Noctalia IPC (`noctalia msg config-reload`), so the shell, terminal and themed apps recolor themselves in a few seconds
3. Pick **Regenerate & apply now** to activate it, **Tune settings** to adjust with live preview, or **Toggle** to disable

It is stamp-cached (same wallpaper + same settings → no work) and safe to leave enabled. Generated palette defaults to `Live.json` (renameable from the menu). Requires `recol` on `PATH` (the hook sets it up itself).

**Save copy**: the **`Save copy (freeze current)`** option snapshots the current live palette into a new one (named after the current wallpaper by default), with overwrite confirmation and an optional "activate now". The copy is frozen: the live palette keeps regenerating on wallpaper changes, the snapshot never changes.

### Features

* **Live palette**: auto-regenerates the palette from the current wallpaper every time Noctalia's wallpaper changes (native `wallpaper_changed` hook, v5+)
* **Community format 2**: palettes are written in the current Noctalia palette format (`"version": 2`, snake_case roles), passing the [community-palettes](https://github.com/noctalia-dev/community-palettes) CI check including WCAG contrast (accents are auto-darkened/brightened until every `on_x` pair reaches ≥ 3.0:1)
* `wallpaper-to-noctalia --migrate [files...]` converts older format 1 palettes (`mPrimary`, `selectionFg`, ...) to format 2 in place, losslessly
* **Live preview** of the palette while you tune (chafa sixel)
* **Dark + light variants**, both calibrated (the light one inverts surfaces but darkens accents to keep contrast)
* **Unified background**: `mSurface` == `terminal.background` — the shell and terminal apps share the exact same color
* **Minimum luminance floor**: very dark wallpapers never generate pitch-black backgrounds
* **Stamp-based cache** (settings hash + image mtime): re-processing a folder skips everything already generated
* **Parallel processing** (`MAX_JOBS=N`, default 4)
* **Palette management**: edit (ENTER), rename (CTRL-R), delete with confirmation (CTRL-D, multi-select with TAB)
* Custom palette names keep their Title Case ("Cream Autumn"), with an overwrite prompt if the name exists
* Settings persist in `~/.config/noctalia-ricer.conf`

### Notes

* Custom palettes are **not** added to `registry.json` — Noctalia detects them by file
* Images with very few unique colors (e.g. `nord_bridge.png`) make recol panic: the script automatically retries via an ffmpeg JPEG re-encode, and with added noise if needed
