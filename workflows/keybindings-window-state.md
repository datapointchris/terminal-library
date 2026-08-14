---
tags: [keybindings, window-state, aerospace, hyprland, tmux, neovim]
---

# keybindings — window state across the window manager, Tmux, Neovim

```text
# AeroSpace (macOS)
alt + f                           toggle floating / tiling
alt + shift + f                   toggle fullscreen (menu bar stays)

# Hyprland (Linux)
SUPER + f                         toggle floating / tiling
SUPER + shift + f                 toggle maximize (waybar and gaps stay)
SUPER + p                         pseudotile: keep own size, centered in tile
SUPER + q                         close window
SUPER + `                         toggle scratchpad
SUPER + shift + '                 move window to scratchpad

# Tmux
prefix + z                        zoom pane (fills the window)
prefix + x                        kill pane
prefix + k                        kill window (confirms first)

# Corne TMUX layer
G + right inner thumb             zoom pane

# Neovim
<leader>wm                        maximize / restore split (vim-maximizer)
```

Hyprland has two fullscreen modes and only one is bound. `fullscreen, 1`
maximizes: the window fills the workspace and waybar and the gaps stay.
`fullscreen, 0` is true fullscreen and covers them, which also strands the other
tiled windows on that workspace behind the bar. Only mode 1 is bound, and that
matches AeroSpace's `fullscreen` — it fills the workspace and leaves the menu
bar up.

Pseudotile is the one that reads as a bug. A pseudotiled window keeps its own
size and sits centered in its tile slot, so it will not grow when the workspace
has room for it. `hyprctl clients` does not report the flag in any field, so the
tell is arithmetic: the window is smaller than its tile with equal space on both
sides. `SUPER + p` again clears it.
