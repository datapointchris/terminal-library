---
tags: [gh, github, pr, pull-request, branch, review, recipe, workflow]
---

# open a PR from the terminal (branch → commits → body worth reading)

```bash
# GOAL: get finished work onto a branch and into a PR whose overview you can
# actually skim later. The diff is often not read. The body always is — so the
# body is the deliverable, not a formality.
#
# GITHUB. For work / Bitbucket Data Center this is `bbkt`:
#   workflows show review-and-merge-a-pr-with-bbkt

# 1. BRANCH — named after the work
git switch -c split-config-loader
                      # plain words, what the work IS. Never an icb id, never a
                      # ticket key: a branch may cover one item, several, or none.

# 2. COMMIT as you go — these are the review units inside the PR
git commit            # conventional, imperative, ≤50-char subject, body = WHY
                      # No stacking layer needed: atomic commits already give
                      # the reviewer (you) something to read one piece at a time.

# 3. PUSH and CREATE
git push -u origin HEAD
gh pr create --fill                  # quick: title+body from the commits
gh pr create --body-file pr.md       # real: an overview you wrote
gh pr create --draft                 # not ready to be read yet
gh pr create --title "..." --body "..."

# 4. WATCH the checks without leaving the terminal
gh pr checks --watch  # blocks until CI settles
gh pr view            # description, commits, checks, comments (current branch)
gh pr view --web=false --comments

# 5. THE STANDARDS PASS — from a session that did not write the code
/audit-pr             # numbered findings, posted as one comment

# 6. FIX and the PR updates itself
git commit && git push               # pushing to the branch updates the PR
gh pr ready                          # draft → ready when it is
```

## What the body has to carry

```text
WHAT THE WORK WAS FOR    one or two lines. Not the diff — the reason.
WHAT YOU DECIDED         and what you rejected. The part that is not
                         recoverable from the code afterwards.
WHAT TO LOOK AT          the two or three places worth your eye. This is the
                         triage, and on a big PR it is the only part that works.
```

Everything else is ceremony. A body that restates the diff is a body nobody
reads twice, and you are the only reader.

## Gotchas

```bash
# - `gh pr create` with no --title/--body opens $EDITOR. --fill takes them from
#   the commits, which is only good if the commit bodies were good.
# - `-u origin HEAD` pushes the current branch under its own name; plain
#   `git push` on a fresh branch errors with no upstream.
# - The PR updates on every push. There is no "resubmit" — fix, commit, push.
# - Opening the PR finishes the work. Merging is a separate act: it deploys.
# - Don't `gh pr review --approve` your own PR. GitHub refuses it, and there is
#   no approval step in this workflow — reading and merging is the whole gate.
# - Draft PRs still run CI. Use --draft for "not ready to read", not to save CI.
```

Related: `ai-review-and-commit` (the loop this is step 7 of),
`review-and-merge-a-pr` (the other end), `gh-dash` (every open PR at once),
`git-conventional-commits` (what makes `--fill` worth using), `review-diff`
(reading the branch before you open it).
