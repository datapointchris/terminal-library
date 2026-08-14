---
tags: [neovim, visual-block, multicursor, vim-visual-multi, sort, filter, columns]
cadence: 2mo
---

# Reshape columns without writing a regex

> A regex is the wrong tool when the shape is positional rather than textual —
> a column to prefix, a run of numbers to renumber, three identical words to
> change. This Lab drills visual block, `g<C-a>`, multiple cursors and the
> external filter, and says plainly which of them your config actually gives
> you. Leader is Space.

## Setup

Copy this into your other pane to stage a scratch dir to play in:

```bash
LAB=$(mktemp -d) && cd "$LAB"
cat > items.txt <<'EOF'
item 0
item 0
item 0
item 0
EOF
cat > wide.txt <<'EOF'
alpha:1:prod
b:22:dev
ccccc:333:prod
EOF
cat > deploys.txt <<'EOF'
web 30
api 100
db 9
api 100
EOF
nvim items.txt
```

Undo with `u` after each step.

## Steps

1. **Prefix every line in a column.** `gg` `<C-v>` `3j` `I- ` `<Esc>`
   - Expect: all four lines gain `- `, after a beat.
   - Why: `<C-v>` is blockwise visual; `I` inserts at the block's left edge on
     every line. The change appears on line 1 only until you press `<Esc>` —
     that pause is normal, not a failure.

2. **Append at ragged line ends.** `u`, then `gg` `<C-v>` `3j` `$A ;` `<Esc>`
   - Expect: a `;` at the end of each line regardless of its length.
   - Why: `$` in blockwise mode means "to the end of each line", not to a
     fixed column. This is the case a regex handles fine (`:%s/$/ ;/`) — worth
     knowing both, because `$A` composes with the block you already have.

3. **Delete a column.** `u`, then `gg` `<C-v>` `3j` `ll` `d`
   - Expect: the first three characters vanish from every line.
   - Why: the block is a rectangle, so `d` removes exactly it. Doing this by
     regex means counting characters into a pattern — positional work that
     `\{3}` can express but nobody can read.

4. **Renumber a column.** `u`, then `gg` `f0` `<C-v>` `3j` `g<C-a>`
   - Expect: the zeros become `1`, `2`, `3`, `4`.
   - Why: `<C-a>` increments a number; the `g` prefix makes it *cumulative*
     down a blockwise selection. There is no regex for this — `\=line('.')`
     numbers by line, not by position in a selection, and breaks the moment
     the run does not start at line 1.

5. **Change three occurrences with cursors instead of a pattern.**
   `:e deploys.txt`, put the cursor on `api`, then `<C-n>` `<C-n>`
   - Expect: the first `api` is selected, then the second — two live cursors.
     `c` now changes both, `<Esc>` `<Esc>` leaves multi-cursor mode.
   - Why: `<C-n>` is vim-visual-multi's find-under-cursor, its one entry point
     that survives in this config. You see each match before it is included,
     which is what a `:%s` cannot give you. Inside the mode, `n` takes the next
     match, `q` skips one, `Q` removes one.

6. **Select every occurrence at once.** `<Esc>`, cursor on `api`, then `\\A`
   - Expect: every `api` in the file gets a cursor.
   - Why: `\\A` is VM's Select-All — two backslashes, because `g:VM_leader` is
     unset and defaults to `\\`. Reach for this when you want to *see* the
     matches; reach for `:%s` when you already trust the pattern.

7. **Align columns with an external filter.** `:e wide.txt`, then
   `:%!column -t -s: -o' '`
   - Expect: three tidy aligned columns.
   - Why: `:[range]!cmd` pipes the lines through a shell command and replaces
     them with its output. This config has no alignment plugin, and it does
     not need one — `column`, `sort`, `jq`, `awk` are all one `:%!` away.
     `u` undoes it like any other edit.

8. **Sort, four ways.** `:e deploys.txt`, then `:sort`, `u`, `:sort n`, `u`,
   `:sort u`, `u`, `:sort! n`
   - Expect: lexical (`api api db web`); numeric by the first number found
     (`db 9`, `web 30`, `api 100`); lexical with duplicates removed; then
     numeric descending.
   - Why: plain `:sort` compares text, so `100` sorts before `9`. `n` finds
     the first number in each line and compares it as a number, `u` dedupes,
     `!` reverses. `:sort /\S* /` sorts by what follows the match, which is
     how you sort on the second field.

## Which tool for which shape

```text
# positional, same edit every line      →  <C-v> block  (I, A, $A, d, c)
# a counting sequence                   →  g<C-a> over a block
# a few occurrences you want to SEE     →  <C-n> / \\A   (vim-visual-multi)
# every occurrence, pattern you trust   →  :%s
# which LINES, not which text           →  :g  /  :v
# a job some other program already does →  :%!column -t   :%!sort   :%!jq .
```

## What is broken in this config, so you do not go looking

```text
<C-S-Down> / <C-S-Up>    intended as VM Add-Cursor-Down/Up in
                         plugins/vim-visual-multi.lua, but core/keymaps.lua
                         binds the same chords to winresize AND loads after
                         lazy — so they resize the window and the VM remap
                         never takes effect. <C-n> and \\A are unaffected.
<C-Down> / <C-Up>        VM's own defaults, deliberately given up to
                         vim-tmux-navigator for pane movement.
```
