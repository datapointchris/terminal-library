---
tags: [digest, synthesis, research, content, cross-analysis, recipe]
---

# synthesize a batch of sources (digest: many URLs → cross-analyzed themes)

```bash
# GOAL: you have a pile of related sources — a YouTube playlist, or a text file
# of article/video URLs on one topic — and you want digest to read each AND
# cross-analyze them into shared themes + novel contributions, then explore the
# result. This is the batch pipeline, start to finish.

# 1. CROSS-ANALYZE the list — reads every source it does not hold, then
#    synthesizes them into one saved analysis
digest synthesize <playlist-url>       # a YouTube playlist, OR...
digest synthesize ./urls.txt           # ...a text file, one URL per line

# 1b. TOO MANY SOURCES for one synthesis? Group them first
digest cluster ./urls.txt              # topic clusters, each saved as pending
digest pending                         # the clusters waiting, by ID
digest synthesize 3                    # cross-analyze cluster 3 on its own

# 2. CHECK where things stand
digest status                          # DB state: pending clusters, unviewed
                                       # items, and the SUGGESTED NEXT ACTION.
                                       # This is your dashboard between steps.

# 3. READ + EXPLORE the synthesis
digest browse                          # fzf over content + analyses, interactively
digest browse --unviewed               # just what you have not read yet
digest show <id>                       # render one analysis via glow
digest search 'query'                  # full-text across everything analyzed

# 4. CALIBRATE — close the loop between your judgment and Claude's
digest rate <id>                       # your own score + optional review
digest deltas                          # where YOUR rating and Claude's quality
                                       # score disagree MOST — the items worth a
                                       # second look. This is the payoff of rating.
```

## The single-source detour

```bash
digest analyze <url> --discuss   # one URL, and open an interactive Claude session
                                 # to dig into it (critical analysis + personal
                                 # connections) instead of just saving a summary.
digest analyze <url> --no-save   # throwaway — analyze without touching the DB.
```

## Gotchas

```bash
# - `analyze` over a list reads each source on its own and cross-analyzes
#   NOTHING. For a topic you want SYNTHESIZED, use `synthesize` — a pile of
#   single analyses never gets cross-linked into themes.
# - `synthesize` summarizes rather than analyzes the sources it reads, because
#   the cross-analysis reads the saved summaries. Run `analyze` over the same
#   list first if you also want each one scored; `synthesize` then skips them.
# - Above --max-sources, `synthesize` says to `cluster` and goes on anyway —
#   one analysis over that many sources says little about any of them.
# - `rate` feels optional but it's what makes `deltas` useful later; rating as
#   you read is what surfaces your blind spots vs Claude's.
```
