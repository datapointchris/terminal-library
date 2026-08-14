---
tags: [neovim, global, ex, batch, normal, macros, registers, filter]
cadence: 1mo
---

# Run a command on every matching line (:g and :v)

> `:s` changes text inside a line. `:g` chooses *which lines* something happens
> to, and the something is any Ex command — including a macro. It is the
> closest thing Vim has to a for-loop, and no plugin in this config replaces
> it. This Lab drills the verbs. Leader is Space.

## Setup

Copy this into your other pane to stage a scratch dir to play in:

```bash
LAB=$(mktemp -d) && cd "$LAB"
cat > deploys.txt <<'EOF'
api      prod  ok
web      prod  fail
worker   dev   ok
db       prod  fail
cache    dev   ok
EOF
nvim deploys.txt
```

Undo with `u` after each step so every step starts from the same five lines.

## Steps

1. **Dry run first.** `:g/fail/`
   - Expect: the two `fail` lines echo at the bottom. Nothing changes.
   - Why: with no command, `:g` defaults to `:p` (print). `:g` is *not*
     covered by `'inccommand'`, so this and `u` are the only safety nets.
     `:g/fail/#` prints them with line numbers instead.

2. **Delete the matches.** `:g/fail/d`
   - Expect: `web` and `db` are gone, three lines left.
   - Why: `:[range]g/pattern/command` runs `command` on every matching line.
     The default range is the whole file, which is why there is no `%`.

3. **Keep only the matches.** `u`, then `:v/fail/d`
   - Expect: the inverse — only `web` and `db` survive.
   - Why: `:v` is `:g!` — run on lines that do *not* match. `v` is for
     inVerse, nothing to do with visual mode. This is the extract idiom, and
     it is how you turn a file into a list of just the interesting rows.

4. **Substitute on matching lines only.** `u`, then `:g/prod/s/ok/OK/`
   - Expect: `api` becomes `OK`; `cache` and `worker` keep lowercase `ok`.
   - Why: a bare `:%s/ok/OK/` would hit the dev rows too. `:g` supplies the
     *which lines* and `:s` the *what changes* — the two questions are
     separate, and jamming both into one pattern is what makes it unreadable.

5. **Copy matches to the end.** `u`, then `:g/fail/t$`
   - Expect: the two `fail` lines are appended at the bottom, originals intact.
   - Why: `t` is `:copy` and `$` is the last line. `:g/fail/m$` would *move*
     them instead. This is how you partition a file without leaving it.

6. **Move matches to the top, and notice the order.** `u`, then `:g/prod/m0`
   - Expect: `db`, `web`, `api` at the top — **reversed** from their original
     order.
   - Why: each match is moved above line 0 in turn, so the last one moved ends
     up first. That is why `:g/^/m0` reverses an entire file, and it is the
     canonical surprise of this command.

7. **Act on the match and its neighbours.** `u`, then `:g/^web/.,+1d`
   - Expect: `web` *and* `worker` are both deleted.
   - Why: inside `:g` the cursor sits on each match, so `.` is that line and
     ranges are relative to it. `.,+1` is "this line and the next".

8. **Run normal-mode keys on every match.** `u`, then
   `:g/dev/normal A  <-- check`
   - Expect: the two `dev` lines gain the trailing text.
   - Why: `:normal` feeds keystrokes. Use `:normal!` to ignore your mappings,
     which you want inside `:g` unless you meant to fire one.

9. **Run a macro on every match.** `u`, record with `qq` `A  # recheck` `<Esc>`
   `q`, then `:g/fail/normal @q`
   - Expect: both `fail` lines get the comment.
   - Why: this is the pairing that matters. `:g` finds the lines and the macro
     does per-line work no single regex can express — anything you can do by
     hand once becomes a batch operation.

10. **Collect matches into a register.** `u`, then
    `:let @a='' | g/fail/y A`, then `:enew` and `:put a`
    - Expect: a new buffer holding just the two `fail` lines, under a blank
      line.
    - Why: the **uppercase** `A` appends; a lowercase `:g/fail/y a` overwrites
      and leaves you only the last match. The blank line is yank-append
      starting from an empty register — `:v/\S/d` clears it.

## Where your config shortcuts it

```text
# It does not, inside one buffer. There is no :g plugin, and that is the
# point of drilling it. What the config adds is the CROSS-FILE cousin:

<leader>fg  →  <C-q>          telescope live-grep, all results to quickfix
:cfdo %s/OLD/NEW/ge | update  the same idea over every file in that list
<leader>fq                    fuzzy-search the quickfix list
]q  /  [q                     walk it in the buffer
<leader>fr                    inspect the register step 10 built

# Rule: inside the file you are looking at, :g is fewer keystrokes and needs
# nothing loaded. Across files, use the quickfix list — you can delete rows
# from it before committing, which :g gives you no chance to do.
```

## The whole thing in one breath

```vim
:g/pattern/            print them — the dry run
:v/pattern/d           keep only the matching lines
:g/pattern/normal @q   run a recorded macro over every match
```
