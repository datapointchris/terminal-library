---
tags: [claude, tmux, git, worktree, watchexec, delta]
---

# following a Claude agent from the next pane

An extra pane beside the one Claude is working in, showing the **code** it is writing. Everything
here is read-only and safe to run while the agent works — except `lazygit`, which is called out at
the bottom.

## 1. Find where it is actually writing

The pane's path lies. `claude` never `cd`s — its Bash calls run in subshells — so
`pane_current_path` shows where the session started, forever. If the agent is working in a worktree,
that path is a clean repo with nothing to see.

```bash
git -C ~/dotfiles worktree list
git -C ~/.dotfiles-wt/<branch> status --short
```

That is the answer nearly every time. See `cards show git-worktrees`.

**Several agents running at once?** Claude Code keeps a live registry:

```bash
jq -r '"\(.name)\t\(.status)\t\(.tmux)\t\(.cwd)"' ~/.claude/sessions/*.json
```

```text
dotfiles-d3   busy   code:@11.%1    /home/chris/dotfiles
refcheck-b2   idle   code:@12.%20   /home/chris/tools/refcheck
```

Two things it will get wrong if read naively. `.cwd` is where the session **started**, so it names
the repo and never the worktree — `worktree list` is still what finds the work. And the window id in
`.tmux` goes **stale** when a pane is moved between windows; only the `%pane` part stays true for
the life of the pane.

## 2. Open the panes

```bash
prefix |      # split right
prefix -      # split down
prefix v      # main-vertical: one tall pane left, the rest stacked right
prefix z      # zoom one pane full-screen and back
```

More in `cards show tmux-commands`.

## 3. Live diff — the code

```bash
WT=~/.dotfiles-wt/<branch>
BASE=$(git -C "$WT" merge-base origin/main HEAD)
GITDIR=$(git -C "$WT" rev-parse --git-dir)

watchexec -c -w "$WT" -w "$GITDIR/index" -- \
  "git -C $WT diff $BASE | delta --paging=never --width=\$COLUMNS"
```

`-c` clears between runs. Read it as: *when anything under the worktree changes, print the whole
diff since the branch point.*

Four things that are wrong if you leave them out:

- **`-w "$GITDIR/index"`** — without it a commit by the agent does not repaint, because watchexec
  ignores VCS directories by default. And it is **not** `$WT/.git/index`, which does not exist in a
  worktree.
- **`--width=$COLUMNS`** — a piped stdout hides the terminal size from delta and it silently falls
  back to 80 columns.
- **`origin/main`, not `main`** — a local `main` is pinned at your last checkout of it. See
  `cards show git-diff-viewing`.
- **Never let the command write into the watched directory.** Output to a file under `$WT` retriggers
  the watch, which retriggers the write; measured at 232 runs from a single edit. Print to the
  terminal.

**`git diff` does not show new files.** An untracked file is invisible no matter what you diff
against — and in agent work the new file is usually the whole point. Read-only way to include them:

```bash
git -C "$WT" ls-files --others --exclude-standard \
  | while read -r f; do git -C "$WT" diff --no-index -- /dev/null "$f"; done
```

Never `git add -N` to make them show up: that writes to the index of a tree the agent is mid-write
in.

## 4. Digging in properly — lazygit **mutates**

```bash
lazygit -p "$WT"
```

Full TUI, delta already configured as its pager, and it shows untracked files without the loop
above. It is the right tool once you want to *read* rather than *watch*.

But it is not passive: `space` stages, `d` discards, and both land in the tree the agent is actively
writing. Use it when the agent is idle, or accept that you are sharing a working tree with it.

## The shape

```text
pane 1   claude, working
pane 2   watchexec + git diff + delta      <- what it wrote
```
