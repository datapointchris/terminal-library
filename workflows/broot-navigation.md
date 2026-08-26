---
tags: [broot, navigation, tui, search, disk-usage, git]
---

# broot — whole-tree search, sizing and navigation

Launch as `br`, never `broot`. `br` is a shell function that passes `--outcmd` and
evaluates what broot writes there, which is the only way a child process can move
the parent shell.

broot never scrolls past what fits. It folds what does not into a `N unlisted` row
and re-expands around whatever you filter for. That is the difference from yazi:
yazi shows one directory, broot shows the whole subtree at once.

`?` or `F1` opens a searchable help screen listing every verb and its key. It is the
fastest answer to "what is this bound to".

## alt-Enter does two different things

The help table binds two verbs to the same key:

| Verb | Key | Applies to | Result |
| --- | --- | --- | --- |
| `cd` | `alt-Enter` | directory | shell ends up in that directory |
| `open_leave` | `alt-Enter` | file | hands the file to `xdg-open`, shell does not move |

So `alt-Enter` on a file quits broot without moving you. `:cd` on a file selection
does nothing at all — it writes no command. To reach the directory a file sits in,
the sequence is `:parent` then `:cd`.

Bind it in `verbs.hjson`, since a sequence cannot be typed as one internal:

```hjson
{
    invocation: cd_parent
    shortcut: cdp
    key: alt-p
    apply_to: file
    cmd_sequence: ":parent;:cd"
}
```

`apply_to: file` keeps it off directories, where `alt-Enter` already works.

## Opening

| Command | Behavior |
| --- | --- |
| `br` | tree at the current directory |
| `br ~/projects` | tree at a path |
| `br -w` | whale spotting: size-sorted, plus hidden, plus gitignored |
| `br -s` / `-d` / `-p` | show sizes / dates / permissions |
| `br -h` / `-i` | include hidden / gitignored files |
| `br -f` | directories only |
| `br -g` | annotate the tree with git status |
| `br --git-status` | show only files with an interesting git status |
| `br --sort-by-date` | most recently modified first, one level |
| `br --sort-by-size` | largest first, one level |
| `br --max-depth 2` | stop descending past a depth |
| `br -c '<cmds>'` | run a command sequence non-interactively |

## Search modes

Type letters to filter. The prefix before the `/` selects the mode, which is why a
bare `foo/` filters nothing — broot reads `foo` as a mode name it does not have.

| Prefix | Searches | Example |
| --- | --- | --- |
| *(none)* | fuzzy on sub path | `flam` matches `src/flag/mod.rs` |
| `f/` | fuzzy on file name | `f/conh` matches `DefaultConf.hjson` |
| `/` | regex on file name | `/rs$` matches `build.rs` |
| `e/` | exact string on file name | `e/feat` matches `help_features.rs` |
| `nt/` | tokens on file name | `nt/fea,he` matches `HelpFeature.java` |
| `ep/` | exact string on sub path | `ep/te\/do` matches `website/docs` |
| `rp/` | regex on sub path | `rp/\d{3}.*txt` matches `dir/a123/b.txt` |
| `t/` | tokens on sub path | `t/help,doc` matches `website/docs/help.md` |
| `c/` | exact string in file content | `c/find(` matches a file containing `a.find(b)` |
| `cr/` | regex in file content | `cr/find/i` matches a file containing `A::Find(b)` |

Tokens mode is the one worth learning. `t/dotfiles,machines` means both tokens
appear somewhere in the path, in any order.

Patterns compose with `!`, `&` and `|`, and parentheses group them:

```text
!md                          everything except paths matching md
(/toml$/|/rs$/)&c/tomato     toml or rs files whose contents match tomato
```

Put the content search last — it is the expensive half.

## Keys and verbs

Type a space or `:` to start a verb, then its name or shortcut.

| Key / verb | Action |
| --- | --- |
| `↑` `↓` | move the selection |
| `Enter` | focus a directory, making it the new root |
| `Enter` on the top line | go up one level |
| `alt-Enter` | cd and quit (directory) / xdg-open and quit (file) |
| `alt-p` | cd to the selected file's directory, if you bound `cd_parent` |
| `Esc` | clear the filter, else revert to the previous state |
| `ctrl-s` | total search — look past the rows broot folded away |
| `ctrl-→` / `ctrl-←` | open or focus a panel to the right or left |
| `ctrl-w` | close the panel without using its selection |
| `:toggle_preview` | open or close the preview panel |
| `:txt` `:img` `:hex` `:tty` | force how the preview renders |
| `:-sd` | apply flags live, here sizes and dates |
| `:cp` / `:mv` | copy or move, prompting for a destination |
| `:cpp` / `:mvp` | copy or move to the other panel |
| `:copy_path` | put the selected path on the system clipboard |
| `:md` | `mkdir -p` a subpath |
| `:trash` `:ot` `:rt` | trash a file, browse the trash, restore from it |
| `:fs` | list mounted filesystems |
| `?` / `F1` | the searchable help screen |
| `:q` | quit without moving the shell |

`:-` is `apply_flags` and takes the same letters as the command line, so `:-hi`
turns on hidden and gitignored without relaunching.

## Configuration lives outside the shell function

`broot --install` patches `.bashrc`, `.bash_profile` and `.zshrc` with a `source`
line. `~/.config/broot/launcher/refused` is what stops it asking, so keep that file
if the `br` function is maintained by hand. `broot --set-install-state installed`
also silences the prompt and writes no rc files.

## Recipes

```bash
# What is eating the disk, drilling in with Enter
br -w ~

# Which subsystem is this change actually in
br --git-status

# Find a config buried in a repo tree, then edit it without leaving
br ~/dotfiles      # type: t/machines,yml   then ctrl-e

# Where does a string appear, and do the hits cluster
br ~/dev           # type: c/broot

# Open a shell wherever the selection is
br                 # select a directory, then ctrl-t
```
