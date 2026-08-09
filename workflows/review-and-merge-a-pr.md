---
tags: [gh, github, pr, pull-request, review, merge, fleet, claude, nvim, ci, recipe]
---

# review and merge a PR (fleet prs → read → merge, no browser)

```bash
# GOAL: take a PR from "haven't looked" to merged without leaving the terminal.
#
# GITHUB. For work / Bitbucket Data Center this is `bbkt`:
#   workflows show review-and-merge-a-pr-with-bbkt

# 1. PICK ONE — from anywhere on the machine, any directory
prs                   # alias for `fleet prs review`
                      #   fzf over every open PR you have, OLDEST FIRST: the
                      #   backlog is a gauge and age is what it measures.
                      #   Esc leaves without doing anything.

# Opens a window `pr <repo> #<n>`, cd'd into that repo, diff in nvim Diffview.
# NO Claude — most PRs are browsed and merged, and a session started for you is
# one you didn't decide you wanted.

# 2. READ IT
#   <Tab> / <S-Tab>   next / previous file
#   <leader>e         the file panel
#   q                 quit Diffview

# 3. MERGE — a separate act, because merging is what deploys
gh pr merge --squash --delete-branch
```

## When it earns a conversation

```bash
prefix |              # split the same window
claude
/review-diff          # the reading protocol: triage first, then per commit
                      #   intent → the 2-4 structural moves → the idiom NAMED.
                      #   Works out its own range from the window it is in.
                      # "explain what I'm looking at" reads your cursor off the
                      #   nvim socket — you point, you don't describe.

/audit-pr             # the standards pass. Run it in a session that did NOT
                      #   write the code: a reviewer holding the author's
                      #   context reproduces the author's blind spots.
```

## Just the list, or the richer TUI

```bash
fleet prs list        # every open PR, oldest first, no picker
fleet prs list --json
gh dash               # sections by state; d diffs, C checks out, m merges
```

## Someone else's PR

```bash
# The verdict verbs only mean something here — on your own PR GitHub refuses a
# self-approval, and a required-approval rule deadlocks a solo repo outright.
gh pr list                       # open PRs in this repo
gh pr view 42                    # description, commits, review state
gh pr checks 42                  # CI — don't review red without knowing why
gh pr checkout 42                # RUN it. Reading says it compiles.
gh pr review 42 --approve
gh pr review 42 --request-changes -b "the retry loop never breaks on 429"
```

## Gotchas

```bash
# - `prs` FETCHES, it does not check out. Your branch and working tree stay put
#   — deliberately, since `gh pr checkout` aborts when the repo is dirty. To run
#   the code: gh pr checkout <n>
# - The diff is three-dot (base...head): from where the branch diverged to its
#   tip, which is exactly what GitHub shows. Never `base..head` — a base branch
#   that moved renders its own commits as changes the PR reverted.
# - `gh pr checkout 42` SWITCHES your branch. `git switch -` (gsw) to come back.
# - Check CI before merging. `gh pr merge` will merge a PR whose checks are
#   still running unless branch protection stops it.
# - A PR in a repo missing from ~/dev/repos.json has no local path, so `review`
#   says so instead of guessing. `fleet prs list` still lists it.
# - `prs` needs tmux — it opens a window.
```

Related: `open-a-pr` (the other end), `gh-dash` (the full TUI),
`ai-review-and-commit` (the loop that fills this queue), `git-diff-viewing`.
