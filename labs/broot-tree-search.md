---
tags: [broot, tree, search, navigation, git, disk-usage]
cadence: 2mo
---

# Search a whole tree with broot

> broot folds a directory tree to fit the screen and re-expands it around whatever
> you filter for. This Lab drills the search modes, the pattern operators, live flag
> toggles, and the `alt-Enter` split that catches everyone — on a throwaway repo.

## Setup

Copy this into your other pane to build a scratch repo and drop into it:

```bash
LAB=$(mktemp -d) && cd "$LAB"
mkdir -p api/handlers api/models web/components docs .cache
printf 'func Authenticate(token string) error {\n' > api/handlers/auth.go
printf 'func GetUser(id int) *User {\n'            > api/handlers/user.go
printf 'type User struct { Token string }\n'       > api/models/user.go
printf 'export function Login() {}\n'              > web/components/Login.tsx
printf '# Docs\nsee the auth flow\n'               > docs/readme.md
printf 'noise\n'                                   > .cache/build.log
printf 'node_modules/\n.cache/\n'                  > .gitignore
mkdir -p node_modules/left-pad && printf 'x\n' > node_modules/left-pad/index.js
for i in $(seq -w 1 30); do mkdir -p vendor/pkg$i; printf 'package p\n' > vendor/pkg$i/mod.go; done
git init -q
for f in api/handlers/*.go api/models/*.go web/components/*.tsx docs/readme.md .gitignore vendor; do git add "$f"; done
git -c user.email=a@b -c user.name=a commit -qm init
# leave the tree dirty so --git-status has something to show
printf 'func Authenticate(token string) error {\n\t// changed\n' > api/handlers/auth.go
printf 'export function Signup() {}\n' > web/components/Signup.tsx
```

## Steps

1. **See the tree fold.** `br`
   - Expect: `vendor` shows a few `pkgNN` rows then `21 unlisted`, and `api` shows
     `2 unlisted`. A `…` after a directory name means its children are folded.
   - Why: broot never scrolls past the screen. It fits the whole hierarchy by
     summarising the part that does not fit, so what you see is always the shape of
     the entire subtree rather than one level of it.
   - Then: `ctrl-s` runs a total search, which reaches into the folded rows.

2. **The default pattern is fuzzy on the *path*.** Type `apiuser`
   - Expect: both `api/handlers/user.go` and `api/models/user.go`, shown with their
     full relative paths.
   - Why: with no prefix, broot matches the letters in order anywhere across the
     whole sub path. `api` comes from the directory, `user` from the filename.
   - Now type `f/apiuser` — expect nothing. `f/` is fuzzy on the *file name* only,
     and no filename contains those letters in order. `Esc` clears the pattern.

3. **Tokens, when you know two words but not the order.** `t/api,user`
   - Expect: the same two files.
   - Why: `t/` requires every comma-separated token to appear somewhere in the path,
     in any order. It is the mode that survives not remembering the layout.
   - Alternative: `nt/` does the same against the file name alone.

4. **Search file contents.** `c/Token`
   - Expect: `api/models/user.go`, with the matching line shown beside it.
   - Why: `c/` is an exact-string content search, and the result keeps the tree. You
     learn *where* the hits cluster, which a flat `rg` list does not tell you.
   - Alternative: `cr/` takes a regex, and `cr/token/i` adds case-insensitivity.

5. **Compose patterns.** `!go` then `(/go$/|/tsx$/)&c/User`
   - Expect: first, everything except the `.go` files. Then only the two `.go` files
     whose contents match `User` — the `.tsx` files are excluded because neither
     contains it.
   - Why: `!` negates, `&` is and, `|` is or, and parentheses group. Put the content
     search last, since it is the expensive half.

6. **Toggle flags without relaunching.** `:-s` then `:-si`
   - Expect: sizes appear in a left column; then `node_modules` appears too.
   - Why: `:-` is the `apply_flags` verb and takes the same letters as the command
     line. `s` sizes, `d` dates, `p` permissions, `h` hidden, `i` gitignored.
   - Note `.cache` stays hidden under `-i` alone — it is a dotfile as well as
     gitignored, so it needs `h` too.

7. **See churn as a shape.** Quit with `:q`, then `br --git-status`
   - Expect: only `api/handlers/auth.go` marked `M` and `web/components/Signup.tsx`
     marked `N`, each in its own branch of the tree.
   - Why: `git status` gives a flat list. On a wide change the tree tells you which
     subsystem the churn is actually in. `br -g` keeps the whole tree and annotates
     it instead of filtering.

8. **The `alt-Enter` split.** `br`, select the `api` directory, press `alt-Enter`
   - Expect: broot quits and your shell is in `api`.
   - Now relaunch, select `docs/readme.md`, press `alt-Enter`. Expect: broot quits,
     the file opens in your desktop handler, and the shell has **not** moved.
   - Why: two verbs share that key. `cd` fires on a directory, `open_leave` fires on
     a file and hands it to `xdg-open`, which in a terminal often shows nothing.
   - The fix is an `edit_leave` verb on `alt-enter` with `apply_to: file`,
     `external: "$EDITOR {file}"` and `leave_broot: true`. Directories keep the
     built-in `cd`, because broot dispatches the key by `apply_to`.
   - Trap: a `cmd_sequence` verb claims its key and then fails to register, leaving
     the key bound to nothing. Keys take `external` or a single `internal`.

## The whole thing in one breath

```bash
br -w ~                    # what is eating the disk, drill in with Enter
br --git-status            # which subsystem is this change in
br                         # then: t/api,user  →  ctrl-e to edit, ctrl-t for a shell
```
