---
tags: [neovim, regex, substitute, global, registers, extract, reshape, recipe]
---

# extract paired values into a list (two lines in, one line out)

```text
# GOAL: a file repeats the same two keys, one per line, always in the same
# order — HostName then UserName, key then value, name then id. You want ONE
# line per pair with the fields reordered: chris@10.0.10.10.
# The trick is that a SEARCH pattern can cross a line break, so a pair is a
# single match and its two halves are two capture groups.

# THE FILE
Host alpha
    HostName=10.0.10.10
    UserName=chris
Host beta
HostName=example.com
UserName=deploy

# 1. WORK ON A COPY — steps 2 and 3 destroy the buffer
:%y                  yank the whole buffer
:tabnew              open a scratch buffer in a new tab
:put                 paste it there; the real file is now untouched

# 2. COLLAPSE EACH PAIR — one substitute, spanning the line break
:%s/\s*HostName=\(\S\+\)\n\s*UserName=\(\S\+\)/\2@\1/
                     \n in a search pattern matches the line break, so both
                     lines are one match. \1 and \2 come back swapped.
                     Leading \s* eats the indent, or it survives into output.

# 3. THROW AWAY EVERY LINE THAT IS NOT A PAIR
:v/@/d               :v = "on lines NOT matching, run this command" → delete.
                     Only the joined lines are left.

# 4. TAKE THE RESULT
<leader>a            select all (gg<S-v>G)
<leader>y            yank it to the system clipboard ("+y)

# THE WHOLE THING
:%y | tabnew | put
:%s/\s*HostName=\(\S\+\)\n\s*UserName=\(\S\+\)/\2@\1/
:v/@/d
```

## What every piece of the pattern does

```text
  \s*         eats the leading indent, zero or more    →  "    "
  HostName=   plain text; = is not special here        →  "HostName="
  \(\S\+\)    one-or-more non-space, captured as \1    →  "10.0.10.10"
  \n          the line break itself                    →  ⏎
  \s*         eats the second line's indent            →  "    "
  UserName=   plain text                               →  "UserName="
  \(\S\+\)    one-or-more non-space, captured as \2    →  "chris"
```

The backslashes are the part that surprises everyone from PCRE. In Vim's
default `magic` level a bare `(` and `+` are **literal characters**, and
`\(` `\+` are the operators. That is inverted from every other regex dialect.

The replacement puts the second capture first, which is the whole point:

```text
  input   line 1   HostName=10.0.10.10
                            └────┬────┘
                                 \1 ─────────────┐
          line 2   UserName=chris                │
                            └─┬─┘                │
                              \2 ────┐           │
                                     ▼           ▼
  output  \2  @  \1           →    chris  @  10.0.10.10
```

## Non-destructive alternative — build the list in a register

```text
:let @a='' | g/HostName=/let @A=matchstr(getline(line('.')+1),'UserName=\zs\S\+')
             \ .'@'.matchstr(getline('.'),'HostName=\zs\S\+')."\n"

# @a       clear it first, or you append to whatever was already there
# @A       UPPERCASE register name = append instead of overwrite
# \zs      "match starts here" — everything before it is context, not result,
#          so matchstr returns the value alone with no capture group needed
# getline(line('.')+1)   the NEXT line, which is where UserName sits

"ap                  paste the list wherever you want it
<leader>fr           telescope: browse the registers and confirm what landed
```

## Gotchas

```text
# - :v/@/d also keeps any pre-existing line that happens to contain @. You are
#   in a scratch copy, so it is visible and one dd away.
# - Other lines BETWEEN the two keys? \n only matches ONE break. Use the
#   non-greedy any-including-newline instead:
:%s/HostName=\(\S\+\)\_.\{-}UserName=\(\S\+\)/\2@\1/
#   \_.  = any character INCLUDING a newline;  \{-}  = non-greedy (PCRE *?)
#
# - Writing \_.*  instead is the silent-wrong-answer trap. Greedy runs to the
#   LAST UserName in the file, so every host collapses into one bogus line
#   pairing the first hostname with the last username. It does not error.
#
# - 'inccommand' previews :substitute as you type it, so you can watch the
#   match before committing. It does NOT preview :g or :v — u is the net there.
```

Pattern syntax in full: neovim-regex-syntax. The `:g` / `:v` verb family:
neovim-global-command. Doing this across many files instead of one:
neovim-refactor-across-files.
