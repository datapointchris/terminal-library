---
tags: [neovim, buffers, windows, keybindings]
---

# neovim buffers, windows, and tabs

**Buffer** — an open file in memory (primary unit of work).
**Window** — a viewport into a buffer (splits).
**Tab** — a layout of windows (like a workspace).

```text
# Buffers (file navigation)
:e file.txt         open file in current window
:ls                 list open buffers
:b name             switch to buffer (partial match works)
:b 3                switch to buffer number 3
gb                  buffer jump menu (snipe)
]b / [b             next / previous buffer (bufferline order)
<leader>fb          find buffer with Telescope
<leader>bd          close buffer, keep the window layout
<leader>bD          close buffer, discard changes

# Windows (splits)
:sp [file]          horizontal split
:vsp [file]         vertical split
Ctrl + ←/↓/↑/→      navigate between splits (vim-tmux-navigator)
Ctrl+Shift + ←/↓/↑/→ resize splits (10 cells, same key as tmux panes)
<leader>w + h/j/k/l resize splits (10 cells)
<leader>wm          maximize / minimize current split
:q                  close current window
:only               close all windows except current

# Tabs (layouts/contexts)
<leader>te          new tab
<leader>tw          close tab
gt / gT             next / previous tab
:tabnew [file]      new tab with optional file
:tabonly            close all tabs except current
```

`<Tab>` is deliberately unmapped. Terminals send one byte for both `<Tab>` and
`<C-i>`, so mapping it would shadow jumplist-forward.

**Edit two files side by side:** `:vsp other-file.txt`
**Quick lookup then return:** `:sp`, look up, `:q`
**Separate contexts:** tabs for different tasks
