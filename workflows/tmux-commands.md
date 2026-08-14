---
tags: [tmux, multiplexer, keybindings]
---

# tmux keybindings — prefix = `Ctrl + Space`

## Panes

| prefix + \|       | split vertical          |
| prefix + -        | split horizontal        |
| Ctrl + ←↓↑→       | navigate panes (vim)    |
| Ctrl+Shift + ←↓↑→ | resize panes            |
| Ctrl + \          | last pane (vim)         |
| prefix + z        | zoom pane (fullscreen)  |
| prefix + x        | close pane (no confirm) |
| prefix + o        | cycle to next pane      |
| prefix + q        | show pane numbers       |
| Alt + { / }       | swap pane back / forward |
| prefix + ;        | toggle last pane        |
| prefix + m        | mark pane (for join)    |

## Reshape (keeps history)

| prefix + !     | break pane → its own window                 |
| prefix + j / J | join a window in as a pane (side / stacked) |
| prefix + v     | layout: big main + stack                    |
| prefix + e     | layout: even split                          |
| prefix + g     | layout: tiled grid                          |
| prefix + Space | cycle layouts                               |

## Windows

| Alt + n / p    | previous / next window         |
| prefix + c     | new window                     |
| prefix + k     | kill window (asks first)       |
| prefix + 0-9   | select by number               |
| prefix + ,     | rename window                  |
| prefix + { / } | swap window left / right       |
| prefix + .     | move window to another session |
| prefix + f     | find window in this session    |

Hold Alt and tap to move several at once. Direction is where the key sits, not
what it stands for: `n` is left of `p`, so Alt+n goes left. This is the reverse
of tmux's own `prefix n` / `prefix p`, which still mean next / previous.

## Sessions

A session is a unit of work — an initiative, not a repo. Its windows are the
places you touch to do it, often different repos, each named for the activity
rather than the directory. Sessions are on the top status line, the focused
session's windows on the second.

| Alt + , / .     | previous / next session          |
| prefix + < / >  | move session left / right        |
| prefix + A / E  | move session to front / end      |
| Alt + o         | last session                     |
| Alt + t         | new session                      |
| prefix + K      | kill session (asks first)        |
| prefix + T      | promote window → its own session |
| prefix + w      | find a window in ANY session     |
| prefix + s      | session picker (sesh)            |
| prefix + L      | last session (sesh)              |
| prefix + d      | detach                           |
| prefix + $      | rename session                   |
| prefix + Ctrl-s | resurrect save                   |
| prefix + Ctrl-r | resurrect restore                |

Both focus axes sit on one modifier and one hand: windows on `Alt + n / p`,
sessions on the two keys below them.

Reordering is ranked by how often each level gets moved. Panes move most, so
they take the root chord `Alt + { / }`. Windows take the prefix braces that
stock tmux gave swap-pane. Sessions move least and take `prefix < / >`, which
window swap vacated.

Every reorder key now lives in the prefix table, which is why none is marked
Linux any more. AeroSpace can only claim a root chord, so moving them there
retired all four collisions at once — `alt-h`, `alt-l`, `alt-a` and `alt-e`.

Front and end get their own keys because reordering is not free: there is no
swap-session in tmux and session ids never change, so a move rebuilds sessions
to reassign them. Crossing the list is one rebuild pass on `a`/`e` against one
per step on `h`/`l`. Windows and panes are carried across intact.

## Copy mode

| prefix + [ | enter copy mode (vi) |
| prefix + P | paste buffer         |
| prefix + y | copy command         |
| prefix + Y | copy directory       |
| v          | begin selection      |
| Ctrl-v     | rectangle toggle     |
| y          | yank selection       |

## Sidebar tree

| prefix + Backspace | tree + focus (quick tree) |
| prefix + Tab       | tree, no focus            |

## Popups & tools

| prefix + t | this reference     |
| prefix + a | Claude popup       |
| prefix + p | pick a PR to review |
| prefix + F | tmux-fzf menu      |

## General

| prefix + R | reload config         |
| prefix + : | command prompt        |
| prefix + I | install plugins (TPM) |
| prefix + U | update plugins (TPM)  |
