---
tags: [gh, gh-dash, github, pr, pull-request, dashboard, tui, review, workflow]
---

# gh dash — every open PR across the fleet, and how stale it is

```bash
# GOAL: one screen answering "am I writing faster than I am reading?". The
# backlog IS the signal — a pile in Stale means code went out that you never
# looked at, which is the thing worth catching before it becomes a habit.

gh dash               # the whole thing. It is a gh extension, not a binary.

# SECTIONS (configured, not built in) — read left to right as triage order
#   Ready     open · yours · not draft · CI green   → can land now
#   Failing   CI red                                → broken, needs you
#   Stale     opened more than 7d ago               → THE GAUGE
#   Draft     not ready to read yet
```

## Keys

```text
MOVING
  j / k  ↓ ↑     down / up the list
  l / h  → ←     NEXT / PREVIOUS SECTION — this is the one you want
  g / G          first / last item in the section
  s              switch the whole view: PRs ↔ Issues
  /              focus the search box (edits this section's query)
  r / R          refresh this section / every section
  q              quit · ? help — the authority when this card drifts

ON THE SELECTED PR
  d              diff it — pages through delta
  e              expand the full description
  C              checkout locally (gh pr checkout)  ← capital C
  c              comment on it                       ← lowercase c
  w              watch checks, with desktop notifications
  m              merge · W draft → ready · u update branch
  x / X          close / reopen
  o              open in browser (you will not need this)
  y / Y          copy the number / the URL
```

## Reading the backlog

```bash
# Stale is the number that matters, not the total. One PR sitting three weeks
# says more than four opened today — count is volume, age is neglect.
#
# The same signal reaches you without opening anything:
doit dashboard        # the `prs` lane: "N open · oldest Xd"
doit next             # the `pr` pursuit, on a 3d cadence, pins when overdue
```

## Gotchas

```bash
# - It is `gh dash`, not `gh-dash`. Installed as a gh extension:
#     gh extension list
# - Config is ~/.config/gh-dash/config.yml, symlinked out of dotfiles.
# - repoPaths is what makes `c` and local actions work, and it is explicit
#   because gh-dash allows only ONE wildcard per owner — these repos live under
#   ~/tools, ~/code, ~/webapps and ~/. Regenerate rather than hand-edit:
#     jq -r '.owner as $o | .repos[] | "    \($o)/\(.name): \(.path)"' \
#       ~/dev/repos.json | sort
# - Sections filter by STATE, not by role. The stock config ships team filters
#   (review-requested:@me, -author:@me) which can never match when every PR is
#   your own — three empty sections and one real one.
# - `status:success` needs checks to have finished. A PR mid-CI shows in neither
#   Ready nor Failing until they settle; that is correct, not a missing PR.
# - `v` approves. It does nothing useful on your own PR — GitHub refuses a
#   self-approval, and there is no approval step in this workflow anyway.
# - C checks out, c comments. They are one shift apart and do very different
#   things; from 3.10.0 both confirm first.
```

Related: `review-and-merge-a-pr` (what to do once you pick one),
`ai-review-and-commit` (the loop that fills this), `open-a-pr`.
