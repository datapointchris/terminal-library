---
tags: [neovim, regex, pattern, substitute, magic, captures, syntax]
---

# neovim regex syntax (what the backslashes mean)

Vim's regex is not PCRE. The difference is not cosmetic: the characters that
are operators and the characters that are literal are **swapped** relative to
every other dialect. `neovim-search-replace` covers the `:s` command forms —
ranges, flags, confirmation. This covers what goes *inside* the slashes.

## The four magic levels — the same pattern, four ways

```text
\v  very magic    \vHostName\=(\S+)      ( + ? { } are operators, like PCRE
\m  magic         HostName=\(\S\+\)      DEFAULT. . * [] are operators;
                                         ( + ? { } need a backslash
\M  nomagic       HostName=\(\S\+\)      only ^ $ stay operators
\V  very nomagic  \VHostName=            only \ is an operator — use for
                                         literal strings full of . and /

# The level applies from where it appears to the end of the pattern, so a
# pattern can start \v and mean it throughout. Default is \m (magic).
```

## Atoms

```text
.        any character except a newline
\s \S    whitespace / non-whitespace
\d \D    digit / non-digit
\w \W    word char [0-9A-Za-z_] / non-word
\a \l \u alphabetic / lowercase / uppercase letter
[abc]    a character class; [^abc] negates it
\<  \>   start / end of a word (a zero-width boundary)
^  $     start / end of LINE (literal anywhere else in the pattern)
\%^ \%$  start / end of FILE
\_s \_d  any class prefixed \_ ALSO matches a newline
\_.      any character INCLUDING a newline — the cross-line wildcard
\n       the line break, in a SEARCH pattern
```

## Quantifiers (magic level: backslash required)

```text
*        0 or more   (the one quantifier that needs no backslash)
\+       1 or more
\?  \=   0 or 1      (both spellings work)
\{n,m}   between n and m
\{-}     0 or more, NON-GREEDY — the PCRE *? equivalent
\{-n,m}  non-greedy, bounded
```

## Groups, captures and alternation

```text
\(...\)  capture group, referred to as \1 … \9 in the replacement
\%(...\) group WITHOUT capturing (PCRE (?:...))
\|       alternation
\zs      "the match really starts here" — everything before is context
\ze      "the match really ends here"  — everything after is context
\&       all branches must match at this position

# \zs replaces most short capture groups. These two are equivalent:
:s/HostName=\(\S\+\)/\1/          capture, then put the capture back
:s/HostName=\zs\S\+//             just refuse to match the prefix
```

## The replacement side

```text
\1 … \9  the capture groups
\0  &    the WHOLE match
~        the previous replacement string
\u \l    uppercase / lowercase the NEXT character
\U \L    uppercase / lowercase until \E
\E       end a \U or \L run
\r       a line BREAK        ← this is the one you want
\n       a NUL byte          ← almost never what you want
\=expr   evaluate a Vimscript expression and use its result
```

```text
# expression replacements — \= turns the right-hand side into code
:%s/^/\=line('.').'. '/           number every line: "1. ", "2. " …
:%s/\d\+/\=submatch(0)*2/g        double every number in the file
:%s/\w\+/\=toupper(submatch(0))/g submatch(0) is the whole match, (1) is \1
```

## The two traps

```text
# 1. \v makes = SPECIAL, and it fails SILENTLY.
:%s/\vHostName=(\S+)/.../         WRONG. In very magic, e= means "optional e",
                                  so the = leaks into the capture and you get
                                  =chris instead of chris. No error is raised.
:%s/\vHostName\=(\S+)/.../        right — escape = under \v

# 2. \n in the REPLACEMENT inserts a NUL, not a line break.
:%s/,/\n/g                        WRONG — writes ^@ characters
:%s/,/\r/g                        right — splits the line
#    Search side and replacement side are different languages. \n means the
#    line break on the left and a NUL on the right.
```

## Seeing it before you commit

```text
/pattern                 search first; 'hlsearch' shows every match
<Esc>                    clear the highlight (mapped to :nohlsearch)
:%s/…/…/gc               c flag = confirm each one: y n a q l
'inccommand' = nosplit   :substitute previews live as you type it.
                         Neovim's default, and it does NOT cover :g or :v.
:set inccommand=split    same preview, plus a split showing every changed line

# 'ignorecase' is OFF here while 'smartcase' is ON, and smartcase only does
# anything when ignorecase is on — so every search is case-sensitive.
\c  \C                   force case-insensitive / case-sensitive, anywhere
                         in the pattern, regardless of the options
```

Reaching a pattern at many lines at once: neovim-global-command. A worked job
end to end: extract-paired-values-into-a-list.
