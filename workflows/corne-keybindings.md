---
tags: [corne, zmk, keybindings, window-manager, tmux, keyboard]
---

# Corne WM and TMUX layers

The two layers that drive the window manager and tmux. The rest of the keymap —
NAV, NPAD, DEVLEFT, ARROW, SYSTEM — is not covered here.

Both layers emit the finished chord. Holding the layer thumb costs two keys
where typing the chord costs three or four, which is the whole reason they
exist.

## Reaching the layers

```text
hold left outer thumb             WM
hold left middle thumb            TMUX     (taps as Backspace)
hold left inner thumb             NPAD     (taps as Delete)
hold G                            NAV      (taps as G)
```

`G` holds NAV — arrows and function keys — not TMUX. The two are easy to
confuse because both put movement on the right-hand home row.

## WM layer — hold the left outer thumb

```text
h/j/k/l         home row          focus window (directional)
y/u/i/o         top row           move window (directional)
n/m/,/.         bottom row        move window into neighbor's group
a/s/d           home row          go to workspace A / S / D
e/x/z                             go to workspace E / X / Z
v                                 go to workspace M
f                                 toggle floating / tiling
r                                 enter the resize mode / submap
'                                 toggle split direction
\                                 toggle layout
bottom-right corner               flatten tree (macOS) / toggle group (Linux)
right inner thumb                 maximize
right middle thumb                toggle scratchpad
right outer thumb                 previous workspace
right pinky, bottom row           a plain Shift
```

The layer carries no shift of its own — the right pinky on the bottom row is a
plain `&kp LSHIFT`, and it composes with any chord on the layer. That is how
move-to-workspace is reached: hold the WM thumb, hold that pinky, tap the
workspace letter.

## One WM layer drives both window managers

The layer is written once against `WMK`, `WMSK` and `WMCK`, which resolve to
`Alt` chords for AeroSpace and `Super` chords for Hyprland. Toggling
`OS_MAC_LAYER` — SYSTEM plus the right inner thumb — swaps the board between
them, and a conditional layer promotes WM to WM_MAC while it is on.

So the two window-manager configs are parallel because one keyboard layer has to
land on both. A binding that exists on only one side has nowhere to sit here.

The scratchpad key is the exception: it emits `Super + grave` literally rather
than through `WMK`, so it reaches Hyprland and does nothing useful on macOS.

## TMUX layer — hold the left middle thumb

```text
h/j/k/l         home row          navigate panes        (Ctrl + arrows)
;                                 last pane             (Ctrl + \)
f + h/j/k/l                       resize panes          (Ctrl+Shift + arrows)
i / o                             previous / next session   (Alt + , / .)
u / p                             move session left / right (prefix < / >)
, / .                             previous / next window    (Alt + n / p)
m / /                             swap window left / right  (prefix { / })
n                                 new window            (prefix c)
e                                 rename window         (prefix ,)
y                                 copy mode             (prefix [)
\                                 split vertical        (prefix |)
'                                 split horizontal      (prefix -)
right inner thumb                 zoom pane             (prefix z)
right middle thumb                session picker        (prefix s)
right outer thumb                 find window, any session (prefix w)
```

Sessions sit on the top row and windows on the bottom, with panes between them
on the home row. Reading down a finger goes session, pane, window rather than
session, window, pane, so the pair worth checking is `i/o` against `,/.`.

`f` is a plain Shift on this layer, on the left index home key. It is the only
left-hand key the layer defines.

## Why Ctrl+Shift carries resize

Ctrl+Shift rather than Ctrl+Alt because on the Corne those are the index and
middle home-row mods, adjacent; Ctrl+Alt was index and ring, skipping the finger
between them. Ghostty binds `ctrl+shift+arrow_left/right` to tab switching by
default, so the Ghostty config unbinds both — without that, resizing left or
right silently switches terminal tabs.

## The keymap is the authority

`config/corne.keymap` in the corne42 ZMK config holds the bindings and
`shared_behaviors.dtsi` holds the macros they expand to. This card is the
annotated view; when the two disagree, the keymap is right.
