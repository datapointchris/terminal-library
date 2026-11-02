---
tags: [neovim, global, ex, batch, filter, registers, normal, macros]
---

# neovim :g and :v (run a command on every matching line)

`:s` changes text inside a line. `:g` picks **which lines** something happens
to, and the something can be any Ex command. It is the closest thing Vim has
to a for-loop, and nothing in the plugin ecosystem replaces it.

```text
:[range]g/pattern/command      on every line matching pattern, run command
:[range]g!/pattern/command     on every line NOT matching  (same as :v)
:[range]v/pattern/command      the inverse. v is for "inVerse", not visual.

# Default range is the whole file, which is why :g needs no % prefix.
# Omit the command and it defaults to :p (print), which is a dry run.
```

## The verbs worth knowing

```text
:g/fail/d              delete every matching line
:v/fail/d              delete every line that does NOT match — i.e. keep
                       only the matches. This is the extract idiom.
:g/^$/d                delete blank lines
:g/prod/s/ok/OK/       substitute, but only on matching lines
:g/fail/t$             copy   matches to the end of the file (t = :copy)
:g/prod/m0             move   matches to the top — and REVERSES them,
                       because each match is moved above the previous one
:g/^/m0                so this reverses the whole file
:g/prod/>              shift matching lines one 'shiftwidth' right
:g/^web/.,+1d          delete the match AND the line after it
                       (the range is relative to each match)
```

## Running normal-mode keys on every match

```text
:g/dev/normal A  <-- dev      append text to the end of each matching line
:g/^api/normal! J             join each match with the line below it
:g/TODO/normal @q             run macro q over every matching line

# normal  vs  normal!  — the ! ignores your mappings, exactly as with any
# other command. Inside :g use normal! unless you MEAN to fire a mapping.
# This pairing is the real power: :g finds the lines, the macro does the
# per-line work that no single regex can express.
```

## Collecting matches into a register

```text
:let @a='' | g/fail/y A       yank each match, APPENDING (uppercase A)
:enew | put a                 paste the collection into a fresh buffer

# Two things bite here:
#   - a lowercase :g/fail/y a  OVERWRITES each time, so you keep only the
#     last match. The uppercase name is the whole mechanism.
#   - starting from an empty register, yank-append leaves a leading blank
#     line. :v/\S/d after pasting clears it, or build the string instead:
:let @a='' | g/fail/let @A=getline('.')."\n"      no leading blank
```

## Measure before you act

```text
:%s/pattern//gn        COUNT matches, change nothing. The n flag reports
                       "3 matches on 3 lines" and makes no edit at all.
:g/pattern/            no command = print every matching line
:g/pattern/#           print them with line numbers

# :s previews live via 'inccommand'. :g does NOT — it is not in the option's
# list. So dry-run with the two above, and keep u in reach.
```

## :g versus the quickfix list

```text
# :g          one buffer, any Ex command, no review step. Instant.
# quickfix    many files, reviewable as a list, driven by :cfdo.
#             <leader>fg → <C-q> → :cfdo … | update
#
# Rule: if the work is inside the file you are looking at, :g is fewer
# keystrokes and needs no plugin. The moment it spans files, you want the
# quickfix list because you can delete rows from it before committing.
# See neovim-refactor-across-files.
```

Pattern syntax for the `/pattern/` half: neovim-regex-syntax.
