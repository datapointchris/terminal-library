---
tags: [git, vcs, rebase]
---

# git rebase — interactive workflow

```bash
# Start interactive rebase
git rebase -i HEAD~3              # rebase last 3 commits
```

Actions (change the word before each commit hash):

| Action | Effect                                          |
| ------ | ----------------------------------------------- |
| pick   | keep commit as-is                               |
| reword | keep commit, edit message                       |
| squash | merge into previous commit, combine messages    |
| fixup  | merge into previous commit, discard this message|
| edit   | pause after commit for amending                 |
| drop   | remove commit entirely                          |

Reorder commits by reordering the lines. Delete a line = drop that commit.

```bash
# During a conflict
git status                        # see which files conflict
# fix the conflicting files, then:
git add <files>
git rebase --continue             # move to next commit
git rebase --abort                # cancel entire rebase
```

Rebase only commits you have not pushed. A rebase gives every commit a new
sha, so rewriting pushed work ends in a force push, and Claude sessions' Bash
hook refuses every force push. A pushed branch catches up with main by merging:

```bash
git switch my-branch
git fetch origin
git merge origin/main -m "chore: merge main into my-branch"
git push                          # no force: a merge only adds commits
```

A conflict in that merge: `unblock-a-pr-that-will-not-merge`.
