---
tags: [git, github, vcs, rebase, merge, conflicts, pull-requests, recipe]
---

# unblock a PR that will not merge (gh → git status → log → rebase → push)

`gh pr merge` refusing and suggesting `--auto` tells you nothing. `--auto` does
not merge — it queues one for whenever the PR becomes mergeable, so it accepts
the command, reports success, and nothing happens. Find out which of the three
states you are actually in first.

```bash
# 1. ASK GITHUB WHY — the step that is always missing
gh pr view <n> --json mergeable,mergeStateStatus
# MERGEABLE   → merge it; the refusal was something else (checks, draft)
# CONFLICTING → real conflict with the base branch; go to 2
# UNKNOWN     → GitHub is still computing after a push. Wait 5s, ask again.

# 2. DOES GITHUB HAVE WHAT YOU HAVE?
git fetch origin && git status
# "have diverged, and have 14 and 13 different commits each"
#   ↑ counts on BOTH sides = you rebased locally and never force-pushed.
#     GitHub is still reading the old commits, so re-rebasing changes nothing.
# "ahead by N" only = a plain push is all you need.
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
# prints the conflicting paths. Optional: rebase tells you the same by
# stopping, one second later and with a tree to clean up if you back out.

# 5. FIX IT
git rebase origin/main
# resolve, then:
git add <file> && git rebase --continue
git push --force-with-lease --force-if-includes
```

Never a bare `--force`. `--force-with-lease` refuses if origin moved since your
fetch; `--force-if-includes` refuses if your ref does not contain what you last
fetched. Together they cannot clobber someone else's push.

```bash
# THE WHOLE THING IN TWO LINES, once you know it is CONFLICTING
git fetch origin && git rebase origin/main   # resolve, git rebase --continue
git push --force-with-lease --force-if-includes
```

**The gotcha**: a conflict is often two people *adding* at the same spot, not
disagreeing. Under `zdiff3` an empty `||||||| base` block proves it — neither
side touched the other's work, so keep both. See `git-merge-conflicts`.
