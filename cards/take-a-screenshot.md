---
tags: [screenshot, screen-capture, print-screen, grim, slurp, wl-clipboard, clipboard, hyprland, aerospace, macos, recipe]
---

# take a screenshot (key to clipboard, clipboard to file)

```text
# GOAL: grab a region or the whole screen, paste it somewhere, or keep it
# as a PNG. Every key puts the image on the CLIPBOARD and saves no file.
# The Corne has no Print key, so the P binds are the ones that always work.

# 1. CAPTURE — same keys on both desks
                         Hyprland (Arch)        AeroSpace (Mac)
region to clipboard      SUPER + shift + p      alt + shift + p
screen to clipboard      SUPER + ctrl + p       alt + ctrl + p
same two, Print key      Print / SUPER + Print  —

# Region: the cursor turns to a crosshair. Drag a box. Esc cancels.
# Forgot the keys? SUPER + shift + / lists every Hyprland bind.

# 2. USE IT — paste straight into Slack, a browser, an issue
Ctrl + v                 (Cmd + v on the Mac)
```

```bash
# 3. KEEP IT — save what is on the clipboard as a file (Arch)
wl-paste --type image/png > ~/shot.png                          # one fixed name
wl-paste --type image/png > ~/shot-$(date +%F-%H%M%S).png       # never overwrites
wl-paste --list-types                                           # image/png = a shot is waiting
```

```bash
# SKIP THE CLIPBOARD — straight to a file from a terminal (Arch)
grim ~/shot.png                          # whole screen
grim -g "$(slurp)" ~/shot.png            # drag a region first

# Same on the Mac
screencapture ~/shot.png                 # whole screen
screencapture -i ~/shot.png              # drag a region first
```

```text
# GOTCHA: the clipboard holds one thing. Copy any text after the shot and
# the image is gone, so save it (step 3) before copying anything else.
# SUPER + v (rofi-clipboard) can still pick an older shot out of history.

# THE WHOLE THING
SUPER + shift + p, drag
wl-paste --type image/png > ~/shot-$(date +%F-%H%M%S).png
```
