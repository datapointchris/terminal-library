---
tags: [neovim, regex, substitute, global, registers, reshape, extract]
cadence: 1mo
---

# Collapse two lines into one (cross-line substitute)

> A config repeats the same two keys in the same order, and you want one line
> per pair with the fields reordered. The whole idea is that a search pattern
> can cross a line break, so a pair is a single match. This Lab drills it, then
> shows the greedy version producing a wrong answer with no error. Leader is
> Space.

## Setup

Copy this into your other pane to stage a scratch dir to play in:

```bash
LAB=$(mktemp -d) && cd "$LAB"
cat > hosts.conf <<'EOF'
Host alpha
    HostName=10.0.10.10
    UserName=chris
Host beta
HostName=example.com
UserName=deploy
EOF
cat > spaced.conf <<'EOF'
Host alpha
    HostName=10.0.10.10
    Port=22
    UserName=chris
Host beta
    HostName=example.com
    IdentityFile=~/.ssh/id
    UserName=deploy
EOF
nvim hosts.conf
```

Goal: turn each `HostName` / `UserName` pair into `chris@10.0.10.10`.

## Steps

1. **Count before you change anything.** `:%s/HostName=//gn`
   - Expect: `2 matches on 2 lines`, and the buffer is untouched.
   - Why: the `n` flag reports the count and suppresses the replacement. This
     is the dry run — it tells you the pattern is right before it edits.

2. **Work on a copy.** `:%y` then `:tabnew` then `:put`
   - Expect: a new tab holding the same text, with a blank first line.
   - Why: the next two steps destroy the buffer. The real file stays open in
     tab 1. `:put` pastes below the cursor, so the empty line 1 stays for now.

3. **Collapse each pair.** `:%s/\s*HostName=\(\S\+\)\n\s*UserName=\(\S\+\)/\2@\1/`
   - Expect: `2 substitutions on 2 lines`, and the pairs become
     `chris@10.0.10.10` and `deploy@example.com`, flush left.
   - Why: `\n` in a *search* pattern matches the line break, so both lines are
     one match. `\(…\)` captures; the replacement writes `\2` before `\1`, which
     is the reorder. The leading `\s*` eats the indent — drop it and alpha's
     line comes out still indented while beta's does not.

4. **Watch it happen next time.** retype the same `:%s` and pause before `<CR>`.
   - Expect: the buffer previews the result live, and `<Esc>` abandons it.
   - Why: `'inccommand'` is `nosplit`, Neovim's default. It previews
     `:substitute` only — `:g` and `:v` get no preview at all.

5. **Keep only the joined lines.** `:v/@/d`
   - Expect: everything else goes, including the blank line from step 2.
   - Why: `:v` runs the command on every line *not* matching. `v` is for
     inVerse. `:g/@/d` would have deleted exactly the lines you wanted.

6. **See the greedy version get it wrong.** `u` back to the pasted copy, then
   `:%s/HostName=\(\S\+\)\_.*UserName=\(\S\+\)/\2@\1/` and `:v/@/d`
   - Expect: one single line, `deploy@10.0.10.10` — beta's user with alpha's
     host. No error, no warning.
   - Why: `\_.` is any character including newline, and `*` is greedy, so the
     match runs from the *first* HostName to the *last* UserName in the file.
     The output is well-formed and wrong, which is the only kind of regex bug
     that survives review.

7. **Handle pairs with lines between them.** `:tabnew spaced.conf`, then
   `:%s/HostName=\(\S\+\)\_.\{-}UserName=\(\S\+\)/\2@\1/` and `:v/@/d`
   - Expect: both pairs join correctly, this time keeping their indent.
   - Why: `\{-}` is the non-greedy quantifier, Vim's spelling of PCRE's `*?`.
     It stops at the *first* `UserName` instead of the last. The indent
     survives because this pattern starts at `HostName`, not at `\s*`.

8. **Build the list without touching the buffer.**

   ```vim
   :let @a='' | g/HostName=/let @A=matchstr(getline(line('.')+1),'UserName=\zs\S\+').'@'.matchstr(getline('.'),'HostName=\zs\S\+')."\n"
   ```

   then `:enew` and `"ap`
   - Expect: the same list, and every original buffer is unmodified.
   - Why: `@A` uppercase appends rather than overwrites — that is the whole
     mechanism, and a lowercase `@a` keeps only the last match. `\zs` sets
     where the match really begins, so `matchstr` returns the value with no
     capture group needed.

## Where your config shortcuts it

```text
<leader>fr    telescope: browse registers, to see what step 8 actually built
<leader>a     select all, then <leader>y to push the result to the clipboard
<leader>u     undo-tree — step 6 is a destructive mistake, and 'undofile' is
              on, so the tree survives closing and reopening the file
<Esc>         clears the search highlight left by 'hlsearch'
```

Nothing here is a plugin. `:g` and `:s` have no plugin equivalent in this
config, which is why they are worth drilling — the alternative is not a
shortcut, it is doing it by hand.

## The whole thing in one breath

```vim
:%y | tabnew | put
:%s/\s*HostName=\(\S\+\)\n\s*UserName=\(\S\+\)/\2@\1/
:v/@/d
```
