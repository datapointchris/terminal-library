---
tags: [git, vcs, worktree, branches]
---

# git worktree — several branches checked out at once

A worktree is a second working directory backed by the same `.git`. Every branch, commit and stash
is shared; only the checkout differs. Reach for one when you need a branch *on disk* without giving
up the branch you are standing on — an agent working while you keep `main`, a long build you do not
want to interrupt, a quick comparison of two branches side by side.

The shared `.git` is the whole point: no second clone, no second fetch, no duplicated objects. In
dotfiles that is 87M of `.git` against a 4.2M worktree.

## The four commands

```bash
git worktree list                      # every checkout of this repo, with branch
git worktree add <path> <branch>       # existing branch into a new directory
git worktree add -b <new> <path>       # create the branch and check it out there
git worktree remove <path>             # delete the directory and deregister it
git worktree prune                     # forget worktrees whose directory is gone
```

`git worktree list` is the one that answers "where is that branch checked out". It works from any
worktree of the repo, because they all share the same `.git`.

```text
/home/chris/dotfiles                              77b3996f [main]
/home/chris/.dotfiles-wt/plugin-manager-providers 77b3996f [plugin-manager-providers]
```

`--porcelain` gives the same thing one field per line, for scripting.

## You do not have to cd — `git -C` runs anywhere

Every git command takes `-C <path>` and behaves as if run from there. That is what makes a worktree
readable from the pane you are already in:

```bash
git -C ~/.dotfiles-wt/some-branch status --short
git -C ~/.dotfiles-wt/some-branch diff origin/main
git -C ~/.dotfiles-wt/some-branch log --oneline -5
```

Assign it once if you are going to type it repeatedly:

```bash
WT=~/.dotfiles-wt/some-branch
git -C "$WT" diff "$(git -C "$WT" merge-base origin/main HEAD)"
```

Nothing stops you `cd`-ing in either, and that is usually simpler — after one visit `z <fragment>`
gets you back. `git -C` earns itself when you want to *stay put*, which is the follow-a-pane case.

## A worktree's `.git` is a file, not a directory

This surprises every script that assumes otherwise:

```bash
cat ~/.dotfiles-wt/some-branch/.git
# gitdir: /home/chris/dotfiles/.git/worktrees/some-branch

git -C ~/.dotfiles-wt/some-branch rev-parse --git-dir
# /home/chris/dotfiles/.git/worktrees/some-branch
```

So `<worktree>/.git/index` does not exist — the index lives under the main repo. Anything watching
for commits or staging has to resolve it with `rev-parse --git-dir` first. Related:
`rev-parse --git-common-dir` gives the *shared* directory (`~/dotfiles/.git`), where the refs and
objects actually are.

## The rules the shared `.git` imposes

**A branch can only be checked out in one worktree.** `git worktree add` refuses a branch already
checked out elsewhere, including in the main repo. That is a feature — it is what makes the shared
`.git` safe — but it means switching a branch between worktrees is a remove-then-add, not a
`git switch`.

**One worktree per stack, at its top.** A stacked branch checked out in a second worktree is skipped
silently by `rebase.updateRefs`, leaving that ref on pre-rebase commits. See
`~/dev/standards/git-workflow.md`.

## In dotfiles specifically, a checkout *is* the running machine

`configs/`, `shell/` and `apps/` are symlinked live into `$HOME`, and the `dotfiles` CLI is installed
editable against `src/`. Checking out a branch in `~/dotfiles` therefore changes the config the
machine runs and the tool that deploys it. Almost no other repo has that coupling, and nothing
announces it.

```bash
git worktree add ~/.dotfiles-wt/<branch> <branch>    # keep ~/dotfiles on main
```

Always run `dotfiles` from `~/dotfiles`, never from inside a worktree — it resolves the repo root
from the CWD when `DOTFILES_DIR` is not exported, and would deploy the worktree's config over the
machine's. Full reasoning: `~/dotfiles/CLAUDE.md` § "The Checked-Out Branch Is Deployed Machine
State".

Most work does not need any of this: commit to `main` and there is no branch to isolate. The
worktree is for the large feature that earns a branch.

## Cleaning up after a squash merge

`git branch -d` **refuses** a branch whose PR was squash-merged. The squash replaced your commits
with one new commit, so git sees no ancestry and reports "not fully merged" even when the trees are
identical. That is not a warning about real unmerged work.

Prove it before forcing, then force:

```bash
git fetch --prune
git log origin/main..<branch>          # empty = nothing missing from main
git diff origin/main <branch>          # empty = trees identical
git worktree remove ~/.dotfiles-wt/<branch>
git branch -D <branch>
```

A merged branch left checked out is also how local `main` silently goes stale: `git rev-list --count
main..<branch>` compares against whatever `main` was at your last fetch, and reports phantom commits
that landed weeks ago.
