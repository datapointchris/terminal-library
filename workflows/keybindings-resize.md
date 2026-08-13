---
tags: [keybindings, resize, aerospace, hyprland, tmux, neovim]
---

# keybindings — resize across the window manager, Tmux, Neovim

```text
# AeroSpace (macOS)
alt + r                           enter Resize Mode
  h/j/k/l                         resize window (directional)
  esc or enter                    exit Resize Mode

# Hyprland (Linux)
SUPER + r                         enter resize submap
  h/j/k/l                         resize window (50px, repeats on hold)
  esc or enter                    exit submap

# Tmux
Ctrl + Shift + ←/↓/↑/→            resize panes (5 cells)

# Neovim
<leader>w + h/j/k/l               resize splits (10 cells)
<leader>wm                        maximize / minimize current split
Ctrl + Shift + ←/↓/↑/→            resize splits (10 cells)
```

Ctrl+Shift+arrows resize Tmux panes and Neovim splits with one binding —
`tmux.conf` checks whether the pane runs vim and forwards the key when it does.

Ctrl+Shift rather than Ctrl+Alt because on the Corne those are the index and
middle home-row mods, adjacent; Ctrl+Alt was index and ring, skipping the finger
between them. Ghostty binds `ctrl+shift+arrow_left/right` to tab switching by
default, so the Ghostty config unbinds both — without that, resizing left or
right silently switches terminal tabs.
