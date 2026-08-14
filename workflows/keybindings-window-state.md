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
maximizes: the window fills the workspace, and waybar and the gaps stay.
`fullscreen, 0` is true fullscreen and covers them, which also strands the other
tiled windows on that workspace behind the bar. Only mode 1 is bound, and that
matches AeroSpace's `fullscreen` — it fills the workspace and leaves the menu bar
up.

Hyprland's third window state, pseudotile, is deliberately bound to no key. A
pseudotiled window holds its whole tile slot but draws at its own stored size,
centered inside it, so it will not grow when the workspace has room. It reads as
a stuck window rather than as a setting. Neither float nor maximize clears it,
and `hyprctl clients` reports the flag in no field — the only tell is arithmetic,
a window smaller than its tile with equal space on both sides. `hyprctl dispatch
pseudo` is the one way into or out of it.
