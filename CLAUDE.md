# terminal-library

Notes on how command-line tools are driven, and the sequences worked out at the
terminal so they are not reassembled from scratch. There is no code here. `doit`
reads every file, and the shapes below are what it parses.

## Three directories answer three questions

- **`workflows/` answers "how do I do this".** A card is what to run once you know
  what you are doing. A reference card holds one tool's commands and keys, and takes
  the tool's name. A recipe card carries one goal through several tools in order. It
  takes the goal's name, verb first, and carries the `recipe` tag.
- **`labs/` drills a tool rather than describing it.** A Lab is worked through on a
  schedule, in a scratch directory, at your own keyboard.
- **`tools/registry.yml` answers "what was that command".** One entry describes one
  thing you can type: a binary, a shell function or a script.

One subject can have a card, a Lab and a registry entry. They are not duplicates. A
second card on a covered topic is sprawl. The card that covers it is extended instead.
Keybindings go on the card of the tool that binds them, never on a card cut by
activity across several tools.

Practice dates, caches and personal lists are state or intentions. They live in the
reader's own config and state directories, never here.

## The reader parses every entry, and one bad file breaks the listing

A card or a Lab opens on front matter, one blank line, then the `# ` title:

```markdown
---
tags: [git, vcs, stash]
cadence: 1mo
---

# git stash — save and restore uncommitted work
```

- **The front matter holds `tags`, plus `cadence` on a Lab.** Nothing else in it is
  read.
- **The title is the first `# ` line.** It is never a front-matter key.
- **Front matter that does not parse as YAML fails `doit workflows list` and
  `doit find` for every card.** No hook catches it. `check-yaml` reads YAML files,
  never the block at the top of a `.md`.
- **Search reads the filename, the title and the tags, never the body.** Every word a
  reader might search for goes in `tags`. Reuse the existing vocabulary, which
  `rg --no-filename '^tags:' workflows labs` prints.
- **Only `*.md` directly inside `workflows/` and `labs/` is read.** A file in a
  subdirectory is invisible.
- **A card is shown through `bat` in a terminal, as raw markdown.** Cards cite each
  other by stem in backticks, as in "See `git-merge-conflicts`". A markdown link
  would print as raw brackets.
- **Every code fence names a language, because markdownlint requires one.** `bash`
  holds commands to run. `text` holds keystrokes, output and pane layouts.

## A filename is an ID, and renaming one breaks what cites it

`doit workflows show`, `doit labs done` and every cross-reference take the stem. A
stem is what `doit workflows new` derives from a title: lowercase, spaces as hyphens,
and nothing but letters, digits and hyphens.

- **Find every citation before renaming, with `rg -n '<old-stem>'`.** Cards cite
  cards by stem, and a registry example cites a card
  (`doit workflows show git-rebase`).
- **A renamed Lab loses its practice history.** The reader keys the last-practiced
  date on the stem, so the renamed Lab reads as never done.
- **`workflows/tmux-commands.md` is read by name.** Its table rows feed the tmux
  keybindings into search. A row counts when its first cell starts with `prefix`,
  `Ctrl` or `Alt`. A literal pipe in a key is written `\|`. Renaming the file,
  writing a key as `C-a`, or putting a column before the key drops those rows
  without an error.

## A Lab is run end to end before it is committed

A Lab opens on a one-line blockquote saying what it drills. `## Setup` stages a
scratch directory, usually starting from `LAB=$(mktemp -d) && cd "$LAB"`.
`## Steps` is a numbered list. Each step is a bold imperative and its command, then
`Expect:` and `Why:` lines. Most Labs close on `## The whole thing in one breath`.

- **Every step is run against the Lab's own Setup block.** A Vim regex can return a
  plausible wrong answer with no error, and only running it shows that.
- **`cadence` is `<n>d`, `<n>w`, `<n>mo` or `<n>y`, or a bare number of days.**
  Anything else reads as zero days. The Lab then comes due again on the day it is
  practiced, and nothing reports the mistake.
- **A Lab without `cadence` is practiced on demand.** It never comes due.

## A registry entry describes a command that exists, spelled the way you type it

The comment block at the top of `tools/registry.yml` defines `requires` and
`installed_via`. Every entry carries `category`, `description`, `installed_via`,
`usage`, `why_use`, `examples` and `tags`. Most also carry `see_also` and
`docs_url`.

- **`usage` starts with what you type.** A key is a package name as often as a
  command, so search takes the invocation from `usage`. The `ripgrep` entry's
  `usage` starts with `rg`.
- **Each example becomes a flashcard.** `doit labs flash` prompts with `desc` and
  reveals `cmd`, so `desc` says what the command does without repeating it.
- **`category` comes from the `categories` list at the foot of the file.** The
  reader groups by whatever an entry says. Nothing checks it against the list.
- **An entry for a tool installed nowhere is deleted.** Saying what you have is the
  registry's whole job, and a full entry for an absent binary says it wrongly. A
  tool that only some machines have keeps its entry, marked as the header describes.
- **Deleting an entry means sweeping every `see_also` that names it.** Those lists
  sit in other entries' blocks.

## The cards are the author's, and a correction changes a fact rather than the voice

The cards are terse, second person and imperative. A code block carries commands
with aligned `#` comments. A table carries keybindings. A recipe numbers its steps,
as in `# 1. ASK GITHUB WHY`, and most recipes close on `## Gotchas`.

- **A key, flag or command goes on a card after it was run, or read from the config
  that binds it.** A key written from memory can name a binding that moved or never
  existed. The card reads just as sure of itself either way.
- **A wrong claim is fixed in place.** The sentences around it stay as written. A
  card that is not wrong is not restyled.
- **A path that differs per machine is written as the command or config key that
  yields it.** Every clone reads the same card. A path from one machine's layout is
  wrong on the next.

## The lint and CI configs are generated, and `.codespellrc` is the one a card edits

`.pre-commit-config.yaml`, `.markdownlint.yaml`, `.editorconfig`, `.shellcheckrc`,
`.github/actionlint.yaml` and `.github/workflows/validate.yml` are generated. Each
says so on its first line. A hand edit is lost at the next regeneration.

`.codespellrc` is this repo's own. A key sequence codespell reads as a typo goes in
its `ignore-words-list`, with a comment naming what the word is. It lands in the
same commit as the card that needs it. `te` is a Neovim binding. `ot` is a broot
shortcut.
