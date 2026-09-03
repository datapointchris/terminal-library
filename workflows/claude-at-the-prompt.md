---
tags: [zsh, claude, ai, keybindings, autosuggestions, atuin, zle, reference]
---

# ask Claude from the zsh prompt, and what else offers you commands

```bash
# FOUR DIFFERENT THINGS OFFER YOU A COMMAND. They are not competing — each one
# knows something the others do not, and they are reached by different keys.
#
#   ghost text      what YOU have run before        free, appears as you type
#   Ctrl-R          what you have run before        a deliberate search, with cwd + exit code
#   Ctrl-X Ctrl-D   what you OWN and have forgotten free, an fzf pick over your own index
#   Ctrl-X Ctrl-A   what you have NEVER run         costs a Claude round trip, ~5s

# ── 1. GHOST TEXT (zsh-autosuggestions) ─────────────────────────────────────
# Gray text ahead of the cursor, drawn from history; falls back to completion
# when history has nothing. Purely passive — it never runs anything.
End        # accept the whole suggestion
Ctrl-E     # same thing (zsh-vi-mode binds ^E to end-of-line)
# Nothing partially accepts: forward-word is unbound here, so Alt-f does not
# take one word. Accept it all and delete back, or keep typing to narrow it.

# ── 2. Ctrl-R (atuin) ───────────────────────────────────────────────────────
# SQLite-backed history search. Richer than the ghost text because it shows when
# you ran it, in which directory, and whether it exited 0.

# ── 3. Ctrl-X Ctrl-D — PICK SOMETHING YOU ALREADY OWN ───────────────────────
# fzf over the federated index: the tool registry, these cards, Claude skills,
# your annotated shell functions, aliases, git aliases, forgit shortcuts and the
# tmux bindings. Enter REPLACES the line with what you would type to run it.
#
# This is the one for "I know I have something for this and I cannot remember
# what". Ctrl-R cannot answer it, because a thing you have never run is not in
# your history — which is most of what you own the week after you write it.
#
# What lands is the row's invocation, not its name: `ripgrep` in the registry
# puts `rg [pattern] [path]` on the line. Placeholders and all — you are meant
# to edit it before pressing Enter, which is the whole reason it goes on the
# line rather than running.

# ── 4. Ctrl-X Ctrl-A — ASK CLAUDE ───────────────────────────────────────────
# Type the English on the prompt line, press it, and the line is REPLACED by the
# command that does it. Nothing executes: read it, edit it, then Enter.
#
#   show the 5 largest files in this directory   →   Ctrl-X Ctrl-A
#   eza -lbF --only-files --sort=size --reverse | head -n 5
#
# The prompt blocks for a few seconds and shows "⋯ asking claude" while it waits.
# It is told the OS and that fd/rg/eza/jq/yq are installed, so what comes back is
# GNU-vs-BSD correct and does not reach for find/grep.

# ── 5. Ctrl-X Ctrl-E — EXPLAIN THIS LINE ────────────────────────────────────
# The inverse. Keeps the line exactly as it is and prints a few lines underneath
# saying what it does, flag by flag, leading with a warning if it is destructive.
# Use it on anything Ctrl-X Ctrl-A hands you that you do not recognize.

# ── SAME THING, WITHOUT A PROMPT LINE ───────────────────────────────────────
doshell find every file over 100MB under my home directory
# Loads the answer at your NEXT prompt via `print -z`, and copies it to the
# clipboard. Use this when you are starting from nothing; use Ctrl-X Ctrl-A when
# you are already halfway through typing and stuck.

dochoose
# The same for the picker: opens it, loads the pick at your NEXT prompt instead
# of replacing the line you are on. Reach for it when what you have already
# typed is worth keeping.

doit choose --source func
# The verb underneath both, in any shell. It prints the invocation and nothing
# else, so `$(doit choose)` composes — bash gets this, just not the keybinding,
# because `print -z` and the ZLE buffer are zsh's.

# ── WHY `?` DOES NOTHING SPECIAL ────────────────────────────────────────────
# atuin ships its own natural-language mode bound to `?` on an empty line. It is
# switched off in ~/.config/atuin/config.toml ([ai] enabled = false) because
# using it means an Atuin Hub account or self-hosting atuin-ai-server, and the
# widgets above already cover it through the Claude subscription. Left on, it
# stole `?` as the first character of a line.
```

## Where this lives

The Claude widgets are defined in the SHELL CONFIG section of `.zshrc`, and the
calls behind them are `doshell_suggest_command` and `doshell_explain_command` in
`shell/common/functions.sh`, shared with `doshell`.

`doit-choose-widget` and `dochoose` are not in dotfiles at all — `doit
shell-widgets zsh` emits them and `.zshrc` caches the block, the same split the
doit completion and startup nudge use. doit is a subprocess and the line editor
belongs to its parent, so the insert is something doit can only emit; putting the
block in dotfiles instead is what broke the last two times, because a config repo
then carries another tool's internals.

All three are bound inside `zvm_after_init`, because zsh-vi-mode wipes the keymap
once the rc file finishes.

Ctrl-X chords rather than Alt: `^[` is vi-cmd-mode, so every Meta binding makes
Escape wait out KEYTIMEOUT before it switches modes. `^X^A`, `^X^E` and `^X^D`
are what the emacs keymap actually leaves free — most of the `^X` space is zsh's
completion-debug and editing defaults, and `^X^K` is `kill-buffer`.
