---
tags: [keybindings, navigation, aerospace, hyprland, tmux, neovim]
---

# keybindings — navigation across the window manager, Tmux, Neovim

```text
# AeroSpace (macOS)
alt + h/j/k/l                     focus window (directional)
alt + shift + h/j/k/l             move window (directional)
alt + a/d/e/m/s/x/z               go to workspace (direct)
alt + shift + a/d/e/m/s/x/z       move window to workspace
alt + tab                         switch to previous workspace

# Hyprland (Linux)
SUPER + h/j/k/l                   focus window (directional)
SUPER + shift + h/j/k/l           move window (directional)
SUPER + a/d/e/m/s/x/z             go to workspace (direct)
SUPER + shift + a/d/e/m/s/x/z     move window to workspace
SUPER + tab                       switch to previous workspace

# Tmux — panes
Ctrl + ←/↓/↑/→                    navigate panes (vim-tmux-navigator)
Ctrl + \                          last pane
Alt + { / }                       swap pane back / forward
prefix + q                        show pane numbers, then type one

# Tmux — windows and sessions
Alt + n / p                       previous / next window
Alt + , / .                       previous / next session
Alt + o                           last session
prefix + { / }                    swap window left / right
prefix + < / >                    move session left / right
prefix + 0-9                      select window by number
prefix + f                        find window in THIS session
prefix + w                        find a window in ANY session
prefix + s                        session picker (sesh + fzf)
prefix + L                        last session (sesh)

# Corne TMUX layer — hold G, everything below is two keys
G + h/j/k/l                       pane left / down / up / right
G + ;                             last pane
G + i / o                         previous / next window
G + u / p                         swap window left / right
G + , / .                         previous / next session
G + m / /                         move session left / right
G + thumbs                        zoom, sesh picker, window finder

# Neovim
Ctrl + ←/↓/↑/→                    navigate splits (vim-tmux-navigator)
gb                                buffer jump menu (snipe)
]b / [b                           next / previous buffer
<leader>fb                        find buffer (Telescope)
:b [name]                         switch buffer by name
gt / gT                           next / previous tab
```

Ctrl+arrows cross Tmux panes and Neovim splits without knowing which side they
land on — vim-tmux-navigator handles the boundary.

Direction is where the key sits, not what it stands for: `n` is left of `p`, so
Alt+n goes left. That inverts tmux's own `prefix n` / `prefix p`.

The TMUX layer emits the finished chord, so holding `G` costs two keys where the
NAV layer's bare arrows cost three or four. Reordering is ranked by how often
each level moves: panes get the root chord, windows the prefix braces, sessions
`prefix < / >`.

`<Tab>` is deliberately not mapped in Neovim. Terminals send the same byte for
`<Tab>` and `<C-i>`, so mapping it would shadow the jumplist-forward key.
