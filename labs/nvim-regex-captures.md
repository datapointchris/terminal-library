---
tags: [neovim, regex, pattern, magic, captures, substitute, syntax]
cadence: 1mo
---

# Vim regex: magic levels, captures and \zs

> Vim's regex is not PCRE, and the difference is not cosmetic — which
> characters are operators and which are literal is *swapped*. This Lab drills
> the syntax on one small file, and spends two steps on the traps that produce
> a wrong answer without raising an error. Leader is Space.

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
printf 'api 3\nweb 10\n' > counts.txt
printf 'a,b,c\n'          > row.csv
nvim hosts.conf
```

Undo with `u` after each step.

## Steps

1. **Look at the matches before you replace them.** `/UserName=\S\+` then `n`
   - Expect: `'hlsearch'` lights up both values; `n` walks them; `<Esc>` clears.
   - Why: a search and a `:s` pattern are the same language, so `/` is a free
     dry run. `:%s/UserName=\S\+//gn` gives the same answer as a count —
     `2 matches on 2 lines` — and changes nothing.

2. **Capture in the default magic level.**
   `:%s/UserName=\(\S\+\)/\1 is the user/`
   - Expect: `chris is the user`.
   - Why: at the default `magic` level, `\(` `\)` are the group operators and
     a bare `(` is a literal paren. `\+` is one-or-more and a bare `+` is a
     literal plus. Inverted from PCRE, and the single biggest tripwire.

3. **The same thing in very magic.**
   `u`, then `:%s/\v(UserName)\=(\S+)/\2 is the \1/`
   - Expect: `chris is the UserName`.
   - Why: `\v` makes every non-alphanumeric special, so `(` and `+` become
     operators and read like PCRE. The level applies from where it appears to
     the end of the pattern.

4. **Now the trap `\v` sets.** `u`, then
   `:%s/\vHostName=(\S+)/[\1]/`
   - Expect: `HostNam[=10.0.10.10]` — not what you asked for, and **no error**.
   - Why: under `\v` the `=` is an operator meaning "zero or one of the
     preceding", so `e=` matched an optional `e` and `(\S+)` then started at
     the `=`. Escape it — `HostName\=` — and it behaves. This is the failure
     shape worth memorising: well-formed, plausible, wrong.

5. **Drop the capture group entirely with `\zs`.**
   `u`, then `:%s/HostName=\zs\S\+/[&]/`
   - Expect: `HostName=[10.0.10.10]`.
   - Why: `\zs` means "the match really starts here". Everything before it is
     context, so `&` — the whole match — is just the value. Most short capture
     groups exist only to put a prefix back, and `\zs` deletes that need.
     `\ze` does the same at the other end: `:%s/\s*\zsHostName\ze=/HOST/`
     replaces the key and leaves the `=` alone.

6. **Change case in the replacement.**
   `u`, then `:%s/UserName=\(\S\+\)/UserName=\U\1/`
   - Expect: `UserName=CHRIS`.
   - Why: `\U` uppercases everything after it until `\E`. `\u` does one
     character — the difference between `Chris` and `CHRIS`.

7. **Run code in the replacement.** `u`, then `:%s/^/\=line('.').'. '/`
   - Expect: every line numbered `1. `, `2. `, `3. ` …
   - Why: `\=` makes the whole right-hand side a Vimscript expression,
     evaluated once per match. `:e counts.txt` and try
     `:%s/\d\+/\=submatch(0)*2/` — `3` becomes `6`, `10` becomes `20`.
     `submatch(0)` is the whole match, `submatch(1)` is `\1`.

8. **The other silent trap: `\n` on the replacement side.**
   `:e row.csv`, then `:%s/,/\n/g`
   - Expect: still one line, now showing `a^@b^@c` — those are NUL bytes.
   - Why: search side and replacement side are different languages. `\n` means
     the line break in a *pattern* and a NUL in a *replacement*. `:%s/,/\r/g`
     is the one that splits the line.

9. **Force case, because smartcase is not doing what it looks like.**
   `:e hosts.conf`, then `:%s/\chostname=/KEY=/g`
   - Expect: both `HostName=` become `KEY=`, despite the lowercase pattern.
   - Why: `\c` forces case-insensitivity anywhere in the pattern and `\C`
     forces sensitivity, both regardless of the options. Worth knowing here
     because this config sets `smartcase` while leaving `ignorecase` off, and
     smartcase only does anything when ignorecase is on — so every search is
     case-sensitive and the option is inert.

## Where your config shortcuts it

```text
'inccommand' = nosplit   :substitute previews live as you type — the fastest
                         way to learn a pattern is to watch it match. Neovim's
                         default, not a setting here. :set inccommand=split
                         adds a pane listing every line it would change.
/pattern  +  hlsearch    see the matches before writing the replacement
<Esc>                    clear the highlight (mapped to :nohlsearch)
:%s/…/…/gc               step through matches: y n a q l
<leader>fh               telescope help tags — :h /\zs, :h /magic, :h sub-replace
```

## The whole thing in one breath

```vim
:%s/pattern//gn              count, change nothing
:%s/KEY=\zs\S\+/NEW/         replace a value without capturing its prefix
:%s/\v(a)\=(b)/\2\1/         very magic — and escape the = or it means "optional"
:%s/,/\r/g                   \r splits a line;  \n writes a NUL
```
