---
tags: [keybindings, window-state, aerospace, hyprland, tmux, neovim, corne]
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
SUPER + shift + `                 stash window in scratchpad

# Corne WM layer — hold the left outer thumb
WM + f                            toggle floating / tiling
WM + right inner thumb            maximize
WM + right middle thumb           toggle scratchpad
WM + shift + right middle thumb   stash window in scratchpad

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

The scratchpad pair sits on one key so the two halves are found together. Show is
`SUPER + \``, stash is the same key with shift. The stash is silent: the window
leaves without focus following it, so an empty scratchpad shows nothing but
`decoration:dim_special` dimming the workspace behind, 0.2 by default. That dim
with no window is what an empty scratchpad looks like, not a failed keybind.

On the Corne the WM layer carries no shift of its own — the right pinky on the
bottom row is a plain `&kp LSHIFT`, and it composes with any chord on the layer.
That is how the scratchpad stash is reached without its own key.

Hyprland's third window state, pseudotile, is deliberately bound to no key. A
pseudotiled window holds its whole tile slot but draws at its own stored size,
centered inside it, so it will not grow when the workspace has room. It reads as
a stuck window rather than as a setting. Neither float nor maximize clears it,
and `hyprctl clients` reports the flag in no field — the only tell is arithmetic,
a window smaller than its tile with equal space on both sides. `hyprctl dispatch
pseudo` is the one way into or out of it.
