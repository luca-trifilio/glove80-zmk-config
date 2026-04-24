# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build

**Local build (Docker):**
```bash
./build.sh          # builds glove80.uf2 in repo root
```
First run is slow (~10-20 min, downloads Nix + ZMK deps). Subsequent runs use the cached Docker image.

**CI build:** push to `main` → GitHub Actions builds automatically → download artifact from Actions tab.

## Flash

1. Hold **Magic key** (bottom-left) while plugging in USB → keyboard enters bootloader, mounts as `GLV80RHBOOT` or `GLV80LHBOOT`
2. Copy `glove80.uf2` via Finder (drag & drop) — macOS blocks terminal writes to the bootloader volume
3. Repeat for the other half

## Architecture

All customization lives in `config/glove80.keymap`. The rest of the repo is build infrastructure.

### Layers
- `Base` (0) — main QWERTY layout with home row mods
- `Lower` (1) — nav, numpad, media; activated via tap-dance on thumb key
- `Magic` (2) — RGB controls, BT pairing, bootloader; hold Magic key
- `Factory` (3) — factory test pattern

### Key behaviors (`behaviors` block)

| Name | Type | Purpose |
|------|------|---------|
| `esc_ctrl` | hold-tap | ESC on tap, LCTRL on hold (left-side only, triggers on right-side keys) |
| `hml` | hold-tap | Home row mod, left hand — hold triggers only for KEYS_RIGHT/THUMBS_RIGHT |
| `hmr` | hold-tap | Home row mod, right hand — hold triggers only for KEYS_LEFT/THUMBS_LEFT |
| `magic` | hold-tap | Magic layer on hold, RGB status on tap |
| `lower` | tap-dance | `mo Lower` on tap, `to Lower` on double-tap |
| `tmux_prefix` | macro | Sends Ctrl+A |

All hold-taps use `flavor = "balanced"`, `tapping-term-ms = <280>`, `quick-tap-ms = <175>`, `require-prior-idle-ms = <150>`, `hold-trigger-on-release`.

### Key position groups
`KEYS_LEFT`, `KEYS_RIGHT`, `THUMBS_LEFT`, `THUMBS_RIGHT` are defined as `#define` macros at the top of the keymap — used to restrict which keys activate hold-tap behaviors (prevents misfires when typing fast on the same hand).
