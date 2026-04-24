---
name: glove80-zmk-workflow
description: This skill should be used when the user asks to build or flash Glove80 ZMK firmware, or to tune hold-tap/home-row-mod behavior in glove80.keymap.
---

## Build

Local build via Docker:
```bash
./build.sh   # produces glove80.uf2 in repo root
```
First run ~10-20 min (downloads Nix + ZMK). Subsequent runs use cached image.

## Flash (macOS)

macOS blocks terminal writes to the Glove80 bootloader volume — use Finder drag & drop only.

1. Hold **Magic key** (bottom-left) + plug USB → mounts as `GLV80LHBOOT` / `GLV80RHBOOT`
2. Drag `glove80.uf2` into the volume via Finder
3. Repeat for other half

Do NOT attempt `cp`, `dd`, or `cat >` — all fail with "Operation not permitted" even with Full Disk Access.

## ZMK hold-tap tuning

**`tap-preferred`**: sends tap (ESC) unless held alone for full `tapping-term-ms`. Bad for modifier use while typing — key combo before timeout sends tap, not hold.

**`balanced`**: sends hold when another key is pressed during hold, regardless of timing. Correct for ESC/Ctrl and home row mods.

### Home row mod parameters (working config)
```c
flavor = "balanced";
tapping-term-ms = <280>;
quick-tap-ms = <175>;
require-prior-idle-ms = <150>;
hold-trigger-on-release;
hold-trigger-key-positions = <KEYS_RIGHT THUMBS_RIGHT>; // or LEFT for right-hand mods
```

`hold-trigger-key-positions` restricts hold activation to the opposite hand — prevents misfires when rolling keys on the same hand. `esc_ctrl` uses the same params as `hml`/`hmr`.

## Layout design

### Decidere su hold-trigger-key-positions

Prima di rimuovere le restrizioni posizionali, analizza i dotfiles:
- `tmux.conf`: qual è il prefix? Coinvolge tasti stesso lato?
- `nvim` keymaps: Ctrl+H/J/K/L (right hand) funzionano già con restrizioni left-mod + right-key
- `zsh` bindkeys: raramente critici per questa decisione

Regola: rimuovi hold-trigger-key-positions solo se la maggioranza delle combo critiche è same-hand. Se sono 1-2 combo problematiche, usa macro dedicate invece — preservi i benefici anti-misfire.

### Thumb cluster

Se un modificatore è sulla home row (es. `S=LALT`), il suo thumb key equivalente è ridondante. Rimpiazzalo con una macro ad alta frequenza.

Pattern efficace: **left thumb macro + right hand command** = flusso naturale per prefix-based tools (tmux, ecc.). Esempio: `tmux_prefix` sul thumb sinistro esterno → mano destra libera per h/j/k/l.

### Modifier placement ergonomics

Non mettere il modifier più usato sul mignolo — è il dito più debole e affatica.
Su macOS: Cmd è il modifier più frequente → va sull'**indice** (F/J).

Layout testato e funzionante:
- `A=Shift  S=Opt  D=Ctrl  F=Cmd` (sinistra)
- `J=Cmd  K=Ctrl  L=Opt  ;=Shift` (destra, specchio)
