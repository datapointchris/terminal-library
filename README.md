# terminal-library

How terminal tools are driven, written down so a sequence worked out once is never reassembled from
scratch. It holds reference cards, practice Labs and a registry of the commands themselves. There is
no code. [doit](https://github.com/datapointchris/doit) reads every file, and the shapes below are
what it parses.

"Terminal" rather than "CLI", because most of what is here is not a binary: tmux bindings, shell
functions, aliases, forgit shortcuts. "Library", because it is curated and written rather than
accumulated. That line keeps caches, exports and `{name: date}` state files out of it.

doit is one reader of it, not its owner. What doit records about you stays in doit's own
directories. Practice dates are state, under `$XDG_STATE_HOME/doit/`. The personal registers
(`pursuits.yml`, `register.yml`, `sources.yml`) are intentions, under `$XDG_CONFIG_HOME/doit/`.

Three neighbors take what does not belong here:

- **dotfiles puts a tool on a machine.** This says what the tool is once it is there.
- **`~/notes/dev/` takes the insight that took effort to understand.** This takes the commands you
  type. The Claude Code skill `capture-note` files there, and `capture-workflow` files here.
- **`terminal-library-fleet` holds the fleet's standards and declarations.** It is a separate repo
  with a different reader: `fleet standards` reads it a rule at a time.

## Starting from scratch

A machine needs doit and nothing else. doit clones this repo on its first run.

```bash
dotfiles packages apply --package doit  # the reader, wherever this box's manifest declares it
doit content status                     # the first run clones; prints the path and what chose it
doit find stash                         # every collection at once; pick a row to open it
doit workflows show git-stash           # one card, named by its filename stem
doit labs due                           # the Labs whose cadence has come round
doit workflows list                     # proves the cards parse: one bad front matter fails it all
```

The clone lands at `$XDG_DATA_HOME/terminal-library/`. `$DOIT_LIBRARY_DIR`, or `library_dir` in
doit's `config.toml`, points doit at another checkout instead. `content status` names which of the
three chose the path, so a machine reading the wrong checkout shows it.

The clone refuses a directory that already exists, holds files and is not a git checkout. Deciding
whose files win is not doit's call. `content status` then prints the `git clone` line to run by
hand. The remote is public, so the clone needs no login.

After the clone, every doit command outside `doit content` starts a `git pull --ff-only` in the
background and never waits for it. Fast-forward only is what makes it safe to write cards in the
checkout doit reads. git refuses to merge and refuses to overwrite a modified file. A half-written
card survives the pull. A failed pull is reported on the next run. `doit content sync` retries it in
the foreground.

It is a repo rather than a synced folder because it has to reach a machine outside the Syncthing
fleet. `git clone` is the one channel that reaches all of them.

## Three directories answer three questions

- **`workflows/` answers "how do I do this".** A card is what to run once you know what you are
  doing. A reference card holds one tool's commands and keys, and takes the tool's name. A recipe
  card carries one goal through several tools in order. It takes the goal's name, verb first, and
  carries the `recipe` tag.
- **`labs/` drills a tool rather than describing it.** A Lab is worked through at your own keyboard,
  in a scratch directory the Lab stages. A `cadence` in its front matter decides when it comes due
  again. A Lab without one is practiced on demand.
- **`tools/registry.yml` answers "what was that command".** One entry describes one thing you can
  type: a binary, a shell function or a script.

One subject can have a card, a Lab and a registry entry. They are not duplicates. A second card on a
covered topic is sprawl, so the card that covers it is extended instead. Keybindings go on the card
of the tool that binds them, never on a card cut by activity across several tools.

The registry is reference rather than declaration. dotfiles is what puts a tool on a machine. Its
shell tree ships the functions and scripts, which is what `installed_via: dotfiles` means. For a
package, dotfiles' `install/packages.yml` says how it installs and the machine's manifest says
whether. Most entries here are functions and scripts no package manager installs, so neither file is
derivable from the other. `yq '.tools[].installed_via' tools/registry.yml | sort | uniq -c` counts
them.

## doit reads every entry, and search sees only the filename, the title and the tags

```bash
doit find --source workflow <term>  # one collection; --help lists them
doit show <name>                    # everything indexed under one name, on one page
doit tools show <entry>             # one registry entry: what it is, why, what to type
doit labs list                      # every Lab and when it next comes due
doit labs done <stem>               # mark a Lab practiced, which starts its cadence again
doit labs flash                     # a recall drill built from the registry's examples
```

- **Search reads the filename, the title and the tags, never the body.** A word that appears only in
  a card's body never finds it.
- **The title is the first `# ` line.** It is never a front-matter key.
- **Only `*.md` directly inside `workflows/` and `labs/` is read.** A file in a subdirectory is
  invisible.
- **Front matter that does not parse as YAML fails `doit workflows list` and `doit find` for every
  card.** No hook catches it. `check-yaml` reads YAML files, never the block at the top of a `.md`.
- **A card is shown through `bat` in a terminal, as raw markdown.** A markdown link prints as raw
  brackets, so cards cite each other by stem in backticks, as in "See `git-merge-conflicts`".
- **`workflows/tmux-commands.md` is read by name.** Its table rows feed the tmux keybindings into
  search. A row counts when its first cell starts with `prefix`, `Ctrl` or `Alt`. A literal pipe in a
  key is written `\|`. Renaming the file, writing a key as `C-a`, or putting a column before the key
  drops those rows without an error.

## An entry is written in the checkout doit reads, then committed and pushed

```bash
doit workflows new rsync         # writes workflows/rsync.md from a template; refuses a stem that exists
doit labs new rsync mirrors      # the same for a Lab, scaffolded with Setup, Steps and cadence: 1mo
doit content edit rsync          # opens an existing card or Lab by its stem
"$EDITOR" "$(doit content path)/tools/registry.yml"  # the registry has no doit verb
doit content status              # what is written and not yet committed
git -C "$(doit content path)" commit -m "docs(workflows): add an rsync card" -- workflows/rsync.md
git -C "$(doit content path)" push  # CI runs the hooks over what the push changed
```

A clone doit makes carries no git hooks. This installs the two stages the config uses:

```bash
pre-commit install --install-hooks -t pre-commit -t commit-msg
```

The commit-msg stage holds a message to conventional commits. CI runs the same hooks on every push
and pull request to `main`. A commit from a checkout without them is still checked, after the fact.

From a Claude Code session, `/capture-workflow` writes a card from what the session worked out, then
commits and pushes it.

A card or a Lab opens on front matter, one blank line, then the `# ` title:

```markdown
---
tags: [git, stash, version-control, workflow]
cadence: 1mo
---

# Shelve work in progress with git stash
```

- **The front matter holds `tags`, plus `cadence` on a Lab.** Nothing else in it is read.
- **Every word a reader might search for goes in `tags`.** Reuse the existing vocabulary, which
  `rg --no-filename '^tags:' workflows labs` prints.
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

`doit workflows show`, `doit labs done` and every cross-reference take the stem. A stem is what
`doit workflows new` derives from a title: lowercase, spaces as hyphens, and nothing but letters,
digits and hyphens.

- **Find every citation before renaming, with `rg -n '<old-stem>'`.** Cards cite cards by stem. A
  registry example cites a card, as in `doit workflows show git-rebase`.
- **A renamed Lab loses its practice history.** doit keys the last-practiced date on the stem, so the
  renamed Lab reads as never done.

## A Lab is run end to end before it is committed

A Lab opens on a one-line blockquote saying what it drills. `## Setup` stages a scratch directory,
usually starting from `LAB=$(mktemp -d) && cd "$LAB"`. `## Steps` is a numbered list. Each step is a
bold imperative and its command, then `Expect:` and `Why:` lines. Most Labs close on
`## The whole thing in one breath`.

- **Every step is run against the Lab's own Setup block.** A Vim regex can return a plausible wrong
  answer with no error. Only running it shows that.
- **`cadence` is `<n>d`, `<n>w`, `<n>mo` or `<n>y`, or a bare number of days.** Anything else reads
  as zero days. The Lab then comes due again on the day it is practiced, and nothing reports the
  mistake.

## A registry entry describes a command that exists, spelled the way you type it

The comment block at the top of `tools/registry.yml` defines `requires` and `installed_via`. Every
entry carries `category`, `description`, `installed_via`, `usage`, `why_use`, `examples` and `tags`.
Most also carry `see_also` and `docs_url`.

- **`usage` starts with what you type.** A key is a package name as often as a command, so search
  takes the invocation from `usage`. The `ripgrep` entry's `usage` starts with `rg`.
- **Each example becomes a flashcard.** `doit labs flash` prompts with `desc` and reveals `cmd`, so
  `desc` says what the command does without repeating it.
- **`category` comes from the `categories` list at the foot of the file.** Nothing checks it against
  the list. `doit tools categories list` counts what the entries say, so a misspelled category shows
  as a row of its own.
- **An entry for a tool installed nowhere is deleted.** Saying what you have is the registry's whole
  job. A full entry for an absent binary says it wrongly. A tool that only some machines have
  keeps its entry, marked as the header describes. `doit kit unresolved` lists the rows this machine
  cannot run.
- **Deleting an entry means sweeping every `see_also` that names it.** Those lists sit in other
  entries' blocks.

## The lint and CI configs are generated, and `.codespellrc` is the one a card edits

`.pre-commit-config.yaml`, `.markdownlint.yaml`, `.editorconfig`, `.shellcheckrc`,
`.github/actionlint.yaml` and `.github/workflows/validate.yml` are generated by forge. Each says so
on its first line. A hand edit is lost at the next regeneration.

`.codespellrc` is this repo's own. A key sequence codespell reads as a typo goes in its
`ignore-words-list`, with a comment naming what the word is. It lands in the same commit as the card
that needs it.
