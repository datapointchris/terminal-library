# terminal-library

A curated body of terminal knowledge, authored by hand.

- `workflows/` — reference cards: one tool's commands, or one procedure across several
- `labs/` — hands-on practice Labs, each with a cadence in its frontmatter
- `tools/` — the tool registry: what each tool is, why to reach for it, what to type

The index is a command rather than a list in this file, because a list here goes stale the
first time a card is added:

```bash
doit workflows list    # every card and what it covers
doit labs list         # every Lab and its schedule status
doit find <term>       # search the cards, tools, functions, aliases and keybindings at once
doit workflows show <card>
```

`tools/registry.yml` is reference, not declaration. What puts a tool on a machine is
dotfiles' `packages.yml`; this says what the thing is once it is there. The two describe
largely different populations — most of what is documented here is a shell function or a
script no package manager installs — so neither is derivable from the other.

"Terminal" rather than "CLI" because most of what is here is not a binary: tmux bindings,
shell functions, aliases, forgit shortcuts. "Library" because it is curated and written,
not accumulated — which is the line that keeps caches, exports and `{name: date}` state
files out of it.

[doit](https://github.com/datapointchris/doit) is one reader of it, not its owner. It
clones this to `$XDG_DATA_HOME/terminal-library/` on every machine, which is also where
it is authored: `doit workflows new`, `doit labs new` and `doit content edit` all write
here.

It is a repo rather than a synced folder because it has to reach a machine outside
the Syncthing fleet, and `git clone` is the one channel that reaches all of them.

No code. Personal registers (`pursuits.yml`, `register.yml`, `sources.yml`) are
intentions rather than knowledge and live in `$XDG_CONFIG_HOME/doit/`.
