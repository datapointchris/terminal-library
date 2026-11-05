---
tags: [tmux, tmux-tree-editor, session, window, pane, multiplexer, keybindings, tui, rearrange, move, join, rename]
---

# tmux-tree-editor — rearrange sessions, windows and panes, applied on close

`prefix + j` opens it in a popup. Sessions, windows and panes sit in columns,
the way yazi shows directories. Cut and paste them where they belong; nothing
reaches tmux until the plan is shown and `y` confirms it. `?` lists every key.

## Move around

| Key | Action |
| --- | --- |
| `j` / `k` | Down / up within a column |
| `h` / `l` | Column left / right |
| `g` / `G` | First / last row |
| `ctrl+d` / `ctrl+u` | Half a screen down / up |
| `]` / `[` | The next / previous session's windows, or the next / previous window's panes |

## Edit (staged, not yet applied)

| Key | Action |
| --- | --- |
| `<Space>` | Select the window or pane under the cursor, move down |
| `x` | Cut the one under the cursor, or everything selected |
| `p` | Paste after the cursor |
| `<Esc>` | Clear the cut and the selection |
| `r` | Rename a session or window, retitle a pane |
| `a` | Add a session, window or pane after the cursor |
| `D` | Mark to kill; again to keep. With a selection, every selected one |
| `M` | Move within its column: `j` / `k` walk it, `<Enter>` drops it, `<Esc>` puts it back |
| `u` | Undo the last edit |
| `R` | Reload from tmux, dropping the edit |

## Apply

| Key | Action |
| --- | --- |
| `w` | Show the plan; `y` applies it |
| `q` | Close. With an edit staged the plan opens: `y` applies and closes, `D` closes without applying, `<Esc>` goes back |

## Where a paste lands decides what it becomes

| Cut | Pasted in | Becomes |
| --- | --- | --- |
| a pane | another window's panes column | a pane there, split along the longer side |
| a pane | the windows column | a window of its own |
| a window | another window's panes column | its panes join that window |
| a window | another session's windows column | a window of that session |
| windows | a session just added with `a` | its windows, in place of its empty default |

Row markers: `→` moved, `✎` renamed, `+` new, `✗` killed, `✂` cut, `●` selected.

```text
# Collapse two windows into one
x on the window, k to the other, l into its panes, p, q, y

# Move a session's windows into a new session below it
a and a name on the session, k back to it, l, <Space> each window,
x, h, j to the new session, p, q, y
```

## Gotchas

- Moving a session earlier rebuilds it and every session after it. tmux cannot
  swap or renumber a session, so a fresh one takes its clients, windows and
  name. Options set on the old session itself are not carried over.
- A retitled pane gets `allow-set-title off`, so the shell stops overwriting
  the title at its next prompt.
- A name tmux changed while the editor was open is left alone. Only what you
  edited is renamed.
