---
tags: [git, github, vcs, merge, conflicts, pull-requests, recipe]
---

# unblock a PR that will not merge (gh → git status → log → merge → push)

`gh pr merge` refusing and suggesting `--auto` tells you nothing. `--auto` does
not merge — it queues one for whenever the PR becomes mergeable, so it accepts
the command, reports success, and nothing happens. Find out which of the three
states you are actually in first.

From a Claude session, `/make-pr-mergeable <n>` runs this whole recipe on the
PR's own branch and comments what it resolved.

```bash
# 1. ASK GITHUB WHY — the step that is always missing
gh pr view <n> --json mergeable,mergeStateStatus
# MERGEABLE   → merge it; the refusal was something else (checks, draft)
# CONFLICTING → real conflict with the base branch; go to 2
# UNKNOWN     → GitHub is still computing after a push. Wait 5s, ask again.
# BEHIND      → (mergeStateStatus) blocks only where the repo demands an
#               up-to-date branch: gh pr update-branch <n> merges the base in

# 2. DOES GITHUB HAVE WHAT YOU HAVE?
git fetch origin && git status
# "ahead by N" only = you merged or committed and never pushed. GitHub is
#   still reading the old head, so merging again changes nothing. Push.
# "have diverged, and have 1 and 1 different commits each"
#   ↑ counts on BOTH sides = the branch moved on GitHub (another box) while
#     you committed here. Merge origin/<branch> in, then push.
```

`A..B` is the whole vocabulary: commits in B that are not in A. Point it at
different pairs and it answers every "what is different" question.

```bash
# 3. SEE WHAT MOVED, AND WHY IT BROKE NOW
git log --oneline origin/main..HEAD          # your work
git log --oneline HEAD..origin/main          # what landed under you
base=$(git merge-base origin/main HEAD)      # where the two last agreed
git log --oneline "$base"..origin/main       # exactly what arrived since

# 4. LOOK BEFORE YOU LEAP — merge in memory, touch nothing
git merge-tree --write-tree --name-only origin/main HEAD
# prints the conflicting paths. Optional: the merge tells you the same by
# stopping, one second later and with a tree to clean up if you back out.

# 5. FIX IT — merge the base in, resolve once
git merge origin/main -m "chore: merge main into <branch>"
# resolve, then:
git add <file> && git merge --continue   # opens the prepared message; save it
git push
```

No force. A merge only adds commits on top of what GitHub holds, so a plain
push fast-forwards the branch, and every sha a review or the PR body cites
still resolves. A rebase replays each commit as a new sha, which is why it
ends in a force push. Claude sessions' Bash hook refuses both.

```bash
# THE WHOLE THING IN TWO LINES, once you know it is CONFLICTING
git fetch origin && git merge origin/main -m "chore: merge main into <branch>"
git push        # once resolved: git add <file> && git merge --continue
```

**The gotcha**: a conflict is often two people *adding* at the same spot, not
disagreeing. Under `zdiff3` an empty `||||||| base` block proves it — neither
side touched the other's work, so keep both. See `git-merge-conflicts`.
