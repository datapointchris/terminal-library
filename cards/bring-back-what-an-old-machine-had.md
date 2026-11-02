---
tags: [safekeep, backup, restore, wsl, snapshot, recover, rebuild, migration, files, recipe]
---

# bring back what an old machine had (safekeep files list to restore --source)

```bash
# GOAL: after moving to a new WSL image (or any rebuilt machine), find the files
# an OLDER snapshot holds that this machine never got back, decide one by one,
# and restore only the keepers. Some were dropped on purpose — skip those.
# Each backup copies only what the machine running it has, so a forgotten file
# is in the old snapshots and in NONE taken since. Those old snapshots are its
# only copy. Needs `safekeep files list` — `safekeep update` if it is missing.

# 1. LIST — every file this machine lacks, one line each, flat
safekeep files list --missing        # grouped under the NEWEST snapshot holding
                                     # each file, headed by that snapshot's label
                                     # A file six dirs deep is still one line
safekeep files list --missing | less -R

# 2. READ — look at one before deciding
safekeep files list --missing --json | jq -r '.[] | "\(.snapshot)  \(.stored)"'
less <stored>                        # its copy on the backup drive — READ ONLY

# 3. RESTORE — one path at a time, from the snapshot it is listed under
safekeep restore --to / --from <snapshot> --source ~/path/to/file
safekeep restore --to / --from <snapshot> --source ~/path/to/dir   # whole dir
                                     # modes come back: 0600 keys, and a ~/.ssh it
                                     # has to create comes back at 0700. Another
                                     # username's home is remapped to this one.
                                     # The listing prints this line for its first
                                     # file — copy it and swap the path.
safekeep restore -n ...              # -n rehearses: names what it would write

# 4. CARRY FORWARD — make the next backup take it
safekeep backup run
safekeep files list --from "$(date +%F)" | rg idea.md
                                     # a date picks today's newest run. A hit means
                                     # a config entry covers the path. No hit means
                                     # none does → safekeep config edit, run again

# THE WHOLE THING IN THREE LINES
safekeep files list --missing
safekeep restore --to / --from <snapshot> --source ~/that/file
safekeep backup run && safekeep files list --from "$(date +%F)" | rg file
```

## Gotchas

```bash
# - A file you dropped on purpose stays in --missing forever; nothing records a
#   "no". Read past it.
# - A file under a dir a symlink manager links (~/.config/nvim) is written
#   THROUGH the link, into the repo it points at. --skip-symlinked skips it.
# - files list exiting 1 means part of a snapshot was unreadable; the paths
#   are on stderr and the list is incomplete. Fix access, list again.
# - Never edit or delete inside the backup drive. Unchanged files are hard links
#   shared by every snapshot, so an edit there changes all of them at once.
# - Restoring a file does not add it to the config. Step 4 is how you find out.
# - A path with spaces needs quotes; the line `files list` prints is pre-quoted.
```
