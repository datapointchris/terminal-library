---
tags: [risky, ai, claude, review, git, commit, branch, pr, recipe, workflow]
---

# ai-review-and-commit — your #1 loop (branch, Claude edits, you read, PR)

```bash
# GOAL: turn a yolo Claude session into a branch of clean conventional commits
# and a PR worth skimming. `risky` = `claude --dangerously-skip-permissions` —
# it edits WITHOUT asking, so reading is not optional: it is the whole loop.

# 1. LAND and BRANCH — before the agent starts, not after
z <repo>              # zoxide jump (cd is aliased to z)
git switch -c session-token-refresh
                      # name it after THE WORK. Never an icb id — branches and
                      # items are deliberately not joined.

# 2. RUN the agent
risky                 # fresh --dangerously-skip-permissions session
risky --resume        # OR pick up an existing one (keeps context)

# 3. CHECKPOINT WHILE IT IS WRITTEN — the step that changes designs
                      # Stop it at each logical unit and read what it just did,
                      #   in the session itself. Correct it NOW: after the PR,
                      #   every fix is rework and you are arguing with a diff.

# 4. SEE the whole of it (never commit blind)
gst                   # git status — the file-level overview
git diff              # full diff — pipes through delta automatically
lazygit               # OR go visual: hunk-level, 'e' to edit, space to stage

# 5. STAGE deliberately — one logical change per commit
ga                    # forgit: fzf-pick files/hunks with live preview
git add -p            # OR raw patch mode
                      # DO NOT `git add -A` — stage explicitly.

# 6. COMMIT conventional
git commit            # feat|fix|docs|chore|refactor|test|perf|ci
                      #   imperative, ≤50-char subject, body for the WHY.
                      # These commits are the review units inside the PR.

# 7. PUSH THE BRANCH and OPEN THE PR
git push -u origin HEAD
gh pr create --fill --web=false   # or --body-file for a real overview
                      # Body says what the work was for, what you decided and
                      # rejected, and WHAT TO LOOK AT. Not a diff restatement.

# 8. THE STANDARDS PASS — in a session that did not write the code
/audit-pr             # fresh Claude. Posts numbered findings as one comment.

# 9. MERGE is a SEPARATE act, taken deliberately
gh pr merge --squash --delete-branch
```

## The whole thing in two lines (the common case)

```bash
z <repo> && git switch -c <work> && risky      # ...agent works, prefix+d to read...
gst → ga → git commit → git push -u origin HEAD → gh pr create
```

## Gotchas

```bash
# - risky skips ALL permission prompts. If you didn't read the diff, you didn't
#   review it. Steps 3 and 4 are the safety you traded away at step 2.
# - Branch BEFORE the agent runs. Branching after means `git switch -c` carries
#   the changes over fine, but you spent the whole session on main not knowing
#   whether you meant to.
# - Step 3 is the one that gets skipped, and it is the only one that can still
#   change the design. Step 4 can only find what is already built.
# - `git diff` shows UNSTAGED only. After `ga`, use `git diff --staged`.
# - Merging is not part of finishing. Opening the PR is where the loop ends;
#   merge deploys, so it waits until you have actually read the thing.
# - Nothing needs CI changes — every generated validate.yml already runs on
#   `pull_request`.
```

Related: `open-a-pr` (step 7 in full — what a body should carry),
`review-and-merge-a-pr` (steps 8-9, and the deep read),
`gh-dash` (how many PRs are waiting — the backlog is the gauge), `forgit-git`
(the `ga`/`gd`/`glo` picker family), `git-conventional-commits` (the message
format), `git-diff-viewing` (delta ranges/word-diff).
