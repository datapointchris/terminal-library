---
tags: [claude, narration, tmux, glow, learning, reference]
---

# narrate a session — follow Claude's reasoning in a second pane, not its diff

```bash
# A narration is a markdown file ONE Claude session writes while ANOTHER does the
# work. Not a status, not a summary, not a log of commands: what was decided, the
# step that solved it, and what the obvious move would have got wrong.
#
# It is written OUT OF BAND by a detached headless session. The working pane never
# pauses for it, never mentions it, and never scrolls for it. The channel is
# one-way — you read, the work continues.
#
#   pane 1   claude, working                   <- what it is doing
#   pane 2   narrate watch                     <- why it is doing it
#
# Entries land about a minute BEHIND the turn they describe. A quiet pane during a
# long turn is normal. A pane quiet across several finished turns is not.
```

## Two things share the name, and they are two halves of one concept

```text
/narrate            the SKILL   ~/.claude/skills/narrate/SKILL.md
                    the standard: what an entry should say, how dense, when it
                    has drifted off its focus. Writes nothing itself.

narrate <verb>      the TOOL    ~/.claude/hooks/narrate
                    the mechanics: creates the file, binds the owning session,
                    splits the pane, reports health, stamps it closed. Judges
                    no words.
```

The skill is read by two different sessions. Yours reads § Starting and § Closing
and calls the tool. The writer's headless session reads § How to write one,
§ Density and § Drifting off focus, and appends the entry.

**Which one you want**: *what should this entry say* is the skill. *Why is nothing
being written* is the tool.

## Starting one — you type this in the working session

```bash
# In the pane where Claude is working. Not in a shell.
/narrate                       # start one over what is about to happen
/narrate <title>               # same, naming it yourself
/narrate back                  # the conversation is already going; record it from here
/narrate back <focus>          # same, but scoped to ONE thread of it
/narrate stop                  # close it

# /narrate splits the reading pane itself. There is nothing to open and nothing
# to configure. It prints the path once and then never mentions the narration
# again — that silence is the design, not a failure.
```

**`back <focus>` scopes the whole file, not just the title.** A session that opened
on a PipeWire bug and ended on git worktrees, narrated `/narrate back 'the git
rebase workflow'`, starts at the git decision and never mentions the speaker.
Everything else is left out rather than summarized.

## It closes itself when the work moves on

```bash
# The writer asks, on every turn: is this still the thread the narration was
# opened for? If not, it closes the file and goes quiet. Nothing is said in the
# working pane, and there is nothing to remember to type.
#
# So a narration that stops filling has usually FINISHED, not broken. Check
# which before assuming:
narrate status                 # "none active" = it closed. rows with [XX] = it broke.
```

## When nothing is appearing — `narrate status`

```bash
~/.claude/hooks/narrate status
```

```text
  [ok] file        /home/chris/dev/narrations/2026-09-17-ypl-go-cli-over-the-api.md
  [ok] owner       ypl-ba in /home/chris/.worktrees/ypl/cli-reads
  [ok] transcript  24985 lines, 783 unnarrated
  [--] entries     3 written, file touched 1m ago, status: live
  [..] writer      running now
```

Every row is a failure that has actually stopped a narration, so `ok` is a claim
that that one is not today's fault.

| Row | What it being wrong means |
| --- | --- |
| `file` | the narration was deleted or moved under the state file |
| `owner` | **the one that bites.** Not a live session means no writer will ever spawn |
| `transcript` | the session's jsonl moved; the writer has nothing to read |
| `entries` | `0 written` with everything else `ok` means it has simply not fired yet |
| `writer` | `running now` means an entry is being written; wait a minute |

**`owner` is the row to read first.** A narration is gated on the session id that
opened it. `/narrate` binds that at creation from the tmux pane, so it is right —
but a narration started some other way leaves it for the first Stop hook to claim,
and Claude Code's conversation-title generator is a headless session that fires
within seconds of any prompt on the machine. If it wins, every real turn is
rejected silently and the file stays empty while nothing anywhere says so.

## Reading it

```bash
~/.claude/hooks/narrate pane    # reopen the reading pane beside this one
~/.claude/hooks/narrate watch   # or run the reader in THIS pane
```

```bash
# The reader follows whichever narration is live, not one file. Open it once and
# leave it — when a narration closes it says so and waits for the next one.
#
# Each entry is rendered with glow as it lands and printed ONCE. It never
# redraws, so scrollback and search keep working.
#
# Scroll back and the text holds still. tmux anchors a copy-mode viewport a fixed
# distance from the bottom, so an appending pane normally slides what you are
# reading upward — tail -f and less +F both do this. The reader checks whether you
# are in copy-mode and holds the entry until you leave, then prints it whole.
# Only the clock on the bottom row keeps moving.

# Reading one that is already closed:
narrate watch --file ~/dev/narrations/2026-09-17-ypl-go-cli-over-the-api.md
glow -p ~/dev/narrations/$(date +%F)-*.md
less +F ~/dev/narrations/$(date +%F)-*.md     # raw, follows like tail -f
```

## Asking about one while it is being written

```bash
# Entries are anchored [n1], [n2] … and the anchor is an address in both
# directions. In the working pane you can say:
#
#   "expand n7"          the entry was too thin
#   "n7 is wrong"        no ambiguity about which claim you mean
#
# And an anchor written into a commit body or an icb item points back at the
# paragraph explaining the change. n-20260917-ypl-go-cli-over-the-api#n7 is stable.
```

## What a narration is not

| It is not | That lives in |
| --- | --- |
| a status | `.planning/status.md` |
| what to do next | `icb` |
| a durable lesson | `~/notes/dev/`, via `capture-note` |
| a draft blog post | genericized on the way out, citing the narration id |

A narration is feedstock. It keeps the real hostnames, real paths and real item
numbers, which is exactly what makes it useless as a publishable artifact and
valuable as a record.

## Everything, in one place

```bash
/narrate [title]                       # start, in the working pane
/narrate back [focus]                  # start over a conversation already going
/narrate stop                          # close

narrate status                         # why is nothing appearing
narrate pane                           # reopen the reading pane
narrate watch [--file <path>]          # read in this pane
narrate start <slug> --title T --focus F   # what /narrate runs underneath
narrate stop                           # what /narrate stop runs underneath

# narrate lives at ~/.claude/hooks/narrate and is not on PATH; the skill calls it
# by absolute path and you rarely need it at all.
```
