---
tags: [git, vcs, stash]
---

# git stash — save and restore uncommitted work

```bash
# Stash current changes (staged + unstaged tracked files)
git stash                         # default message
git stash push -m "wip: auth"     # descriptive message

# Stash options
git stash -u                      # include untracked files
git stash -a                      # include untracked + ignored
git stash push path/to/file       # stash specific files only

# Restore stashed changes — run `git stash list` first and name the entry
git stash pop stash@{2}           # apply specific stash + remove from list
git stash apply stash@{2}         # apply specific stash, keep in list

# Browse stashes
git stash list                    # show all stashes
git stash show                    # diff summary of latest
git stash show -p                 # full diff of latest
git stash show stash@{1} -p      # full diff of specific stash

# Clean up — name the entry; there is no safe bare form
git stash drop stash@{2}          # remove specific stash

# Create branch from stash
git stash branch new-branch stash@{2}  # create branch, apply stash, drop it
```

## Gotchas

- **`refs/stash` is one stack shared by every worktree of a repo.** A bare `pop`, `apply` or `drop`
  acts on `stash@{0}`, which is whoever stashed last, possibly in another worktree. Run
  `git stash list` and name the entry you made.
- **`git stash clear` deletes every entry, including other worktrees' stashes.** Drop entries one
  at a time by name instead.
- **To discard local edits, prefer `git restore -- <paths>`.** It touches only this worktree and
  never the shared stack.
- **Untracked files are not stashed without `-u`.** A plain stash leaves them in place, so the
  tree is not what it was at the last commit.
