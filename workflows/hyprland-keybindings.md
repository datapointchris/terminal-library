---
tags: [hyprland, keybindings, window-manager, wayland, linux]
---

# Hyprland keybindings — mod = `SUPER`

The Linux half of a deliberately parallel pair. Every chord here has the same
shape as the AeroSpace one on macOS, with `SUPER` where AeroSpace uses `alt` —
see `aerospace-keybindings.md` for the other side.

## Windows and focus

```text
SUPER + h/j/k/l                   focus window (directional)
SUPER + shift + h/j/k/l           move window (directional)
SUPER + ctrl + h/j/k/l            move window into neighbour's group
SUPER + ctrl + g                  toggle group
SUPER + q                         close window
SUPER + f                         toggle floating / tiling
SUPER + shift + f                 toggle maximize (waybar and gaps stay)
SUPER + drag left mouse           move window
SUPER + drag right mouse          resize window
```

## Workspaces

```text
SUPER + a/d/e/m/s/x/z             go to named workspace
SUPER + shift + a/d/e/m/s/x/z     move window to named workspace
SUPER + 9                         go to workspace 9 (the spare output)
SUPER + shift + 9                 move window to workspace 9
SUPER + tab                       previous workspace
SUPER + `                         toggle scratchpad
SUPER + shift + `                 stash window in scratchpad (silent)
SUPER + ctrl + w                  toggle the spare HDMI output, parking 9 on it
```

Workspaces are named rather than numbered, so the letter is the workspace. `B`
is defined in the config but commented out.

## Layout

```text
SUPER + '                         toggle split direction (dwindle)
SUPER + \                         toggle dwindle / master layout
SUPER + r                         enter resize submap
  h/j/k/l                         resize by 50px, repeats on hold
  esc or enter                    leave the submap
```

`SUPER + '` is a `layoutmsg`, not a top-level dispatcher — dwindle owns the
split toggle, so the binding has to address the layout rather than the
compositor. `SUPER + \` shells out to `hyprctl keyword general:layout` and flips
the value it reads back, because Hyprland has no toggle dispatcher for layouts.

## Applications and system

```text
SUPER + Return                    terminal (Ghostty)
SUPER + Space                     launcher (rofi -show drun)
SUPER + v                         clipboard history (rofi-clipboard)
SUPER + shift + /                 show the live keybind list (rofi-keybinds)
SUPER + shift + r                 reload config
Print                             screenshot a region to the clipboard
SUPER + Print                     screenshot the screen to the clipboard
```

`rofi-keybinds` reads `hyprctl binds` and renders the `bindd` descriptions, so
it is the live list and this card is the annotated one. When the two disagree,
the config is right and this card is stale.

## Media and brightness

```text
XF86AudioRaiseVolume / Lower      volume +/- 5% (repeats on hold)
XF86AudioMute                     mute output
XF86AudioMicMute                  mute microphone
XF86AudioPlay / Prev / Next       playerctl transport
XF86MonBrightnessUp / Down        brightness +/- 10% (repeats on hold)
```

## Fullscreen is bound to mode 1, not mode 0

Hyprland has two fullscreen modes and only one is bound. `fullscreen, 1`
maximizes: the window fills the workspace, and waybar and the gaps stay.
`fullscreen, 0` is true fullscreen and covers them, which also strands the other
tiled windows on that workspace behind the bar. Only mode 1 is bound, and that
matches AeroSpace's `fullscreen` — it fills the workspace and leaves the menu bar
up.

## The scratchpad stash is silent

The scratchpad pair sits on one key so the two halves are found together. Show is
``SUPER + ` ``, stash is the same key with shift. The stash is silent: the window
leaves without focus following it, so an empty scratchpad shows nothing but
`decoration:dim_special` dimming the workspace behind, 0.2 by default. That dim
with no window is what an empty scratchpad looks like, not a failed keybind.

## Pseudotile is bound to no key

Hyprland's third window state, pseudotile, is deliberately bound to no key. A
pseudotiled window holds its whole tile slot but draws at its own stored size,
centered inside it, so it will not grow when the workspace has room. It reads as
a stuck window rather than as a setting. Neither float nor maximize clears it,
and `hyprctl clients` reports the flag in no field — the only tell is arithmetic,
a window smaller than its tile with equal space on both sides. `hyprctl dispatch
pseudo` is the one way into or out of it.
