---
tags: [git, vcs, branches, pull-requests]
---

# git stacking PRs — branch-on-branch workflow

Create a stack: `main → feature-a → feature-b → feature-c`

```bash
git switch main
git switch -c feature-a
# ... work, commit ...
git push -u origin feature-a

git switch -c feature-b           # branches off feature-a
# ... work, commit ...
git push -u origin feature-b
```

Open PRs: `feature-a → main`, `feature-b → feature-a`

```bash
# When feature-a gets updated (review feedback)
git switch feature-a
# ... make changes, commit, push ...

# Merge the fix up into feature-b, then each child above it the same way
git switch feature-b
git merge feature-a -m "chore: merge feature-a into feature-b"
git push                          # no force: every sha in the stack stays

# When a moved main forces a catch-up: into the bottom branch, then up
git switch feature-a
git fetch origin && git merge origin/main -m "chore: merge main into feature-a"
git push                          # then merge feature-a up into feature-b

# When feature-a merges to main
gh pr merge <a> --merge --delete-branch
# Deleting feature-a retargets feature-b's PR onto main. feature-b already
# holds feature-a's commits, so nothing needs catching up.
gh pr edit <b> --base main        # only if feature-a's branch was kept
```

**Keep stacks shallow** — 2-3 deep max.
**Merge from bottom up** — feature-a first, then feature-b.
**Merge up, never rebase** — a rebase rewrites every sha above it and ends in
a force push, which Claude sessions' Bash hook refuses.
**A held-back parent holds its child** — to free the child, cherry-pick its
commits onto a fresh branch off `origin/main` and open a new PR.
