---
tags: [tmux, session, multiplexer, keybindings]
---

# tmux sessions — mental model and persistence

**Session** — a unit of work (an initiative, not a repo), lives in RAM.
**Window** — one place you touch to do that work, named for the activity.
**Pane** — a split within a window (two things visible at once).

One initiative usually spans several repos, so the same repo can hold a window
in more than one session. The window name carries the activity; the session
name is what tells two `homelab` windows apart.

| Component       | What it is                    | Survives reboot? |
| --------------- | ----------------------------- | ---------------- |
| Running session | Process in tmux server (RAM)  | NO               |
| Resurrect state | ~/.local/share/tmux/resurrect | YES              |

Continuum saves every minute, window names and pane contents included, and a
shutdown saves once more before anything is stopped. When the server starts,
the last whole save comes back. So after a reboot, plain `tmux` brings
everything back, including sessions you were done with.

Each pane gets its program back. Claude resumes its own conversation with its
own flags. abtop, btop, yazi, watch and `prs tui` start again, plus what
`~/.config/tmux/restored-programs.conf` adds on that machine. Anything else,
ssh included, comes back typed at the prompt, waiting for Enter.

```bash
# Key bindings
prefix + d              # detach (session keeps running)
prefix + s              # session tree (x kills one, t tags, X kills the tagged)
prefix + L              # last session (instant toggle)
prefix + Ctrl-s         # manual resurrect save
prefix + Ctrl-r         # manual resurrect restore
```

**Always detach, never exit.** Detach = session keeps running in background. Exit = session destroyed. Workflow: open tmux once, live inside it, detach when done.

```bash
# Sessions missing after reboot?
tmux-snapshots list                     # newest saves, whole or cut off, and which is last
tmux-snapshots restore <stamp>          # bring an older one back into the running server
tmux show-options -g | rg continuum     # verify it's enabled
```

A program started as a pane's own process, with no shell under it, comes back
as an empty shell. Start it from the prompt and it restarts.
