# terminal-library

How terminal tools are driven, written down so a sequence worked out once is never reassembled from
scratch. It holds reference cards, practice Labs and a registry of the commands themselves. There is
no code. [howto](https://github.com/datapointchris/howto) reads every file, and the shapes below are
what it parses.

"Terminal" rather than "CLI", because most of what is here is not a binary: tmux bindings, shell
functions, aliases, forgit shortcuts. "Library", because it is curated and written rather than
accumulated. That line keeps caches, exports and `{name: date}` state files out of it.

howto is a reader of it, not its owner, and records nothing about you. A practice schedule or
anything else personal lives in whichever tool keeps it, in that tool's own config and state.

A dotfiles repo puts a tool on a machine. This says what the tool is once it is there.

## Starting from scratch

A machine needs howto and nothing else. howto clones this repo on its first run.

```bash
uv tool install --index https://pypi.ichrisbirch.com/simple/ howto  # the reader
howto library status                    # the first run clones; prints the path and what chose it
howto find stash                        # every collection at once; pick a row to open it
howto cards show git-stash              # one card, named by its filename stem
howto labs list                         # every Lab, with its title and tags
howto cards list                        # proves the cards parse: one bad front matter fails it all
```

The clone lands at `$XDG_DATA_HOME/terminal-library/`. `$HOWTO_LIBRARY_DIR`, or `library_dir` in
howto's `config.toml`, points howto at another checkout instead. `library status` names which of
the three chose the path, so a machine reading the wrong checkout shows it.

The clone refuses a directory that already exists, holds files and is not a git checkout. Deciding
whose files win is not howto's call. `library status` then prints the `git clone` line to run by
hand. The remote is public, so the clone needs no login.

After the clone, every howto command outside `howto library` starts a `git pull --ff-only` in the
background and never waits for it. Fast-forward only is what makes it safe to write cards in the
checkout howto reads. git refuses to merge and refuses to overwrite a modified file. A half-written
card survives the pull. A failed pull is reported on the next run. `howto library sync` retries it
in the foreground.

It is a repo rather than a synced folder because `git clone` reaches every machine, including one
that shares nothing else.

## Three directories answer three questions

- **`cards/` answers "how do I do this".** A card is what to run once you know what you are doing.
  A reference card holds one tool's commands and keys, and takes the tool's name. A recipe card
  carries one goal through several tools in order. It takes the goal's name, verb first, and carries
  the `recipe` tag.
- **`labs/` drills a tool rather than describing it.** A Lab is worked through at your own keyboard,
  in a scratch directory the Lab stages. A Lab carries no schedule.
- **`tools/registry.yml` answers "what was that command".** One entry describes one thing you can
  type: a binary, a shell function or a script.

One subject can have a card, a Lab and a registry entry. They are not duplicates. A second card on a
covered topic is sprawl, so the card that covers it is extended instead. Keybindings go on the card
of the tool that binds them, never on a card cut by activity across several tools.

The registry is reference rather than declaration. A dotfiles repo is what puts a tool on a machine.
Its shell tree ships the functions and scripts, which is what `installed_via: dotfiles` means. For a
package, the machine's manifest says whether it installs there. Most entries here are functions and
scripts no package manager installs. `yq '.tools[].installed_via' tools/registry.yml | sort | uniq
-c` counts them.

## howto reads every entry, and search sees only the filename, the title and the tags

```bash
howto find --source card <term>     # one collection; --help lists them
howto show <name>                   # everything indexed under one name, on one page
howto tools show <entry>            # one registry entry: what it is, why, what to type
howto labs list                     # every Lab, with its title and tags
howto labs show <stem>              # one Lab, to read while you work in another pane
howto labs flash                    # a recall drill built from the registry's examples
```

- **Search reads the filename, the title and the tags, never the body.** A word that appears only in
  a card's body never finds it.
- **The title is the first `# ` line.** It is never a front-matter key.
- **Only `*.md` directly inside `cards/` and `labs/` is read.** A file in a subdirectory is
  invisible.
- **Front matter that does not parse as YAML fails `howto cards list` and `howto find` for every
  card.** No hook catches it. `check-yaml` reads YAML files, never the block at the top of a `.md`.
- **A card is shown through `bat` in a terminal, as raw markdown.** A markdown link prints as raw
  brackets, so cards cite each other by stem in backticks, as in "See `git-merge-conflicts`".
- **`cards/tmux-commands.md` is read by name.** Its table rows feed the tmux keybindings into
  search. A row counts when its first cell starts with `prefix`, `Ctrl` or `Alt`. A literal pipe in a
  key is written `\|`. Renaming the file, writing a key as `C-a`, or putting a column before the key
  drops those rows without an error.

## An entry is written in the checkout howto reads, then committed and pushed

```bash
howto cards new rsync            # writes cards/rsync.md from a template; refuses a stem that exists
howto labs new rsync mirrors     # the same for a Lab, scaffolded with Setup and Steps
howto library edit rsync         # opens an existing card or Lab by its stem
"$EDITOR" "$(howto library path)/tools/registry.yml"  # the registry has no howto verb
howto library status             # what is written and not yet committed
git -C "$(howto library path)" commit -m "docs(cards): add an rsync card" -- cards/rsync.md
git -C "$(howto library path)" push  # CI runs the hooks over what the push changed
```

A clone howto makes carries no git hooks. This installs the two stages the config uses:

```bash
pre-commit install --install-hooks -t pre-commit -t commit-msg
```

The commit-msg stage holds a message to conventional commits. CI runs the same hooks on every push
and pull request to `main`. A commit from a checkout without them is still checked, after the fact.

A card or a Lab opens on front matter, one blank line, then the `# ` title:

```markdown
---
tags: [git, stash, version-control, workflow]
---

# Shelve work in progress with git stash
```

- **The front matter holds `tags`.** Nothing else in it is read.
- **Every word a reader might search for goes in `tags`.** Reuse the existing vocabulary, which
  `rg --no-filename '^tags:' cards labs` prints.
- **Every code fence names a language, because markdownlint requires one.** `bash` holds commands to
  run. `text` holds keystrokes, output and pane layouts.
- **A key, flag or command goes on a card after it was run, or read from the config that binds it.**
  A key written from memory can name a binding that moved or never existed. The card reads just as
  sure of itself either way.
- **A path that differs per machine is written as the command or config key that yields it.** Every
  clone reads the same card. A path from one machine's layout is wrong on the next.

The cards are terse, second person and imperative. A code block carries commands with aligned `#`
comments. A table carries keybindings. A recipe numbers its steps, as in `# 1. ASK GITHUB WHY`, and
most recipes close on `## Gotchas`. A wrong claim is fixed in place. The sentences around it stay as
written.

## A filename is an ID, and renaming one breaks what cites it

`howto cards show`, `howto labs show` and every cross-reference take the stem. A stem is what
`howto cards new` derives from a title: lowercase, spaces as hyphens, and nothing but letters,
digits and hyphens.

- **Find every citation before renaming, with `rg -n '<old-stem>'`.** Cards cite cards by stem. A
  registry example cites a card, as in `howto cards show git-rebase`.
- **A renamed Lab drops off any practice schedule kept for it.** A tool that schedules practice keys
  it on the stem, in its own config and state, so those entries are renamed with it.

## A Lab is run end to end before it is committed

A Lab opens on a one-line blockquote saying what it drills. `## Setup` stages a scratch directory,
usually starting from `LAB=$(mktemp -d) && cd "$LAB"`. `## Steps` is a numbered list. Each step is a
bold imperative and its command, then `Expect:` and `Why:` lines. Most Labs close on
`## The whole thing in one breath`.

- **Every step is run against the Lab's own Setup block.** A Vim regex can return a plausible wrong
  answer with no error. Only running it shows that.

## A registry entry describes a command that exists, spelled the way you type it

The comment block at the top of `tools/registry.yml` defines `installed_via`. Every entry carries `category`, `description`, `installed_via`, `usage`, `why_use`, `examples` and `tags`.
Most also carry `see_also` and `docs_url`.

- **`usage` starts with what you type.** A key is a package name as often as a command, so search
  takes the invocation from `usage`. The `ripgrep` entry's `usage` starts with `rg`.
- **Each example becomes a flashcard.** `howto labs flash` prompts with `desc` and reveals `cmd`, so
  `desc` says what the command does without repeating it.
- **`category` comes from the `categories` list at the foot of the file.** Nothing checks it against
  the list. `howto tools categories list` counts what the entries say, so a misspelled category
  shows as a row of its own.
- **An entry for a tool installed nowhere is deleted.** Saying what you have is the registry's whole
  job. A full entry for an absent binary says it wrongly. A tool that only some machines have
  keeps its entry, marked as the header describes.
- **Deleting an entry means sweeping every `see_also` that names it.** Those lists sit in other
  entries' blocks.

## The lint and CI configs are generated, and `.codespellrc` is the one a card edits

`.pre-commit-config.yaml`, `.markdownlint.yaml`, `.editorconfig`, `.shellcheckrc`,
`.github/actionlint.yaml` and `.github/workflows/validate.yml` are generated. Each says so on its
first line. A hand edit is lost at the next regeneration.

`.codespellrc` is this repo's own. A key sequence codespell reads as a typo goes in its
`ignore-words-list`, with a comment naming what the word is. It lands in the same commit as the card
that needs it.
