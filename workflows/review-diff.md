---
tags: [review, claude, ai, git, diff, diffview, nvim, tmux, learning, recipe, workflow]
---

# review-diff — read the code you didn't type (diff left, Claude right)

```bash
# GOAL: close the comprehension loop. Committing straight to main means nothing
# ever brings you back to code the agent wrote. This is the thing that does.
# It is NOT a bug hunt — `/code-review` does that. This one is for understanding.

# THE WHOLE THING
review-diff           # from inside any repo: opens a `review` window, two panes
prefix + r            # same, without leaving the pane you're in (prefix = C-Space)

#   left  = nvim + DiffviewOpen on the range
#   right = claude, already started on /review-diff <range>
#   C-Left / C-Right cross between them (vim-tmux-navigator)

# WHAT RANGE IT PICKS
review-diff           # everything since your last `menu review done review-diff`
review-diff working   # uncommitted changes — the pre-commit case
review-diff HEAD~5..HEAD    # an explicit range
review-diff a3f2c1          # one commit
review-diff range           # just print what it would review, then exit

# THE WHOLE PR, before you merge it
review-diff "$(git merge-base main HEAD)..HEAD"
                      # merge-base, not main..HEAD — a main that moved since you
                      # branched otherwise drags unrelated commits into the read.

# WHILE A SESSION IS STILL WRITING IT — no second Claude
review-diff solo      # working tree in Diffview, own window, nvim ALONE
prefix + d            # same, from anywhere
                      #   The Claude already running answers `review-diff here`
                      #   off the same per-repo socket. That is the point: you
                      #   ask the session that WROTE it, not a fresh one.

# WHILE YOU'RE READING — point at the screen instead of describing it
#   In the Claude pane, say "explain what I'm looking at" / "this hunk".
#   Claude runs `review-diff here`, which asks the review nvim over its socket
#   for file:line under your cursor, then reads the real file for context.
review-diff here      # file:line under the cursor (what Claude calls)

# CLOSE THE LOOP
review-diff mark      # advances the watermark; next run starts here
```

## What the session actually does

```text
1. triage   one table: read-closely vs skim, with the reason. You pick.
2. walk     intent → the 2-4 structural moves → the language idiom, named
3. quiz     OFF by default. Say "quiz me" to turn it on, then it stops dead
            after each question until you answer.
4. harvest  idiom you didn't know   → capture-note  (~/notes/dev/)
            something that looks wrong → capture-item (an icb item)
            a decision worth keeping   → .planning/status.md
```

## Gotchas

```bash
# - It never offers to fix what it finds. That's deliberate: a fix offer puts you
#   back in delegation mode, which is the habit the review exists to interrupt.
#   Findings become items.
# - The watermark is the script's own, in ~/.local/state/review-diff — one
#   append-only JSONL PER MACHINE, because Syncthing replicates whole files and
#   two boxes appending to one produce .sync-conflict copies. A read takes the
#   newest entry across every machine's file, so a review done on the Mac counts
#   at the Arch desk. Keyed on the git remote, not the path: $HOME differs
#   between the two and a path key makes every repo look unreviewed after a desk
#   switch — syncing correctly and silently doing nothing.
# - `review-diff mark` is the watermark. It is NOT the doit cadence entry: those
#   are two acts against two stores, and marking one does not move the other.
# - `solo` opens nvim with NO Claude beside it, on purpose. If nothing is
#   already running in the repo, `review-diff here` has a socket but you have
#   nobody to ask — use plain `review-diff working` for that.
# - Both panes closing takes the window with it. Quit nvim and exit Claude and
#   you're back where you were.
# - No `jq`, or nothing marked yet → falls back to a 7-day window, and says which
#   it chose on stderr.
```

Related: `ai-review-and-commit` (the loop this plugs into), `open-a-pr` and
`review-and-merge-a-pr` (the gate this reads through), `git-diff-viewing`
(delta ranges/word-diff), `claude-at-the-prompt` (the other ways in).
