# Lutgen Ricer

An interactive Bash TUI for recoloring wallpapers with [lutgen](https://github.com/ozwaldorf/lutgen-rs). Pick a palette, tune the look with a live preview, and batch-process whole folders in parallel.

## Features

- **Live preview** in the terminal (sixel via `chafa`) while you tune settings
- **Palette sync** from [ezequielgk/Palettes](https://github.com/ezequielgk/Palettes), plus all of lutgen's built-in palettes
- **Dark / Light / Both** variants generated in one run
- **Fine-grained control**: algorithm, shape, nearest colors, Hald CLUT level, luminosity, color strength, saturation, contrast, brightness
- **Palette color picker** and palette size limit (All / 8 / 4 / 2)
- **Folder or individual** image processing (jpg, jpeg, png, webp)
- **Parallel jobs** and a stamp cache, so unchanged images are skipped on re-runs
- **Backtracking** with `ESC` at every step

## Dependencies

`lutgen`, `fzf`, `chafa`, `jq`, `curl`, `tar`, and ImageMagick (`magick` or `convert`).

## Usage

Run it from the directory that contains your wallpapers:

```bash
chmod +x ricer.sh
./ricer.sh
```

Then follow the menu: choose a source, tune settings against a sample image, pick the variant(s), and let it run.

Output goes to `<palette>/<folder>/<dark|light>/`.

## Configuration

| Variable | Default | Description |
|---|---|---|
| `MAX_JOBS` | `4` | Number of parallel jobs (`MAX_JOBS=8 ./lutgen-ricer.sh`) |
| `XDG_CONFIG_HOME` | `~/.config` | Settings and palettes live in `$XDG_CONFIG_HOME/lutgen/` |
| `XDG_CACHE_HOME` | `~/.cache` | Skip-cache lives in `$XDG_CACHE_HOME/lutgen-ricer/` |

Settings are saved to `~/.config/lutgen/ricer.conf`.

## Credits

Written by Ezequiel (Zeke). New features by Mikuri12.
