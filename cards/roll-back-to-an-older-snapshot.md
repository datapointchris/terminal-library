---
tags: [safekeep, backup, restore, snapshot, rollback, revert, undo, recover, history, fzf, recipe]
---

# roll back to an older snapshot (safekeep snapshots to restore --from to backup run)

```bash
# GOAL: choose one snapshot out of every snapshot on the drive, put its files
# back on this machine, then back up so that state is the NEWEST snapshot and
# every later run carries on from it. Nothing is pruned: the runs in between
# stay on the drive. A few missing files, one at a time, is
# `bring-back-what-an-old-machine-had`.

# 1. PICK — read every snapshot, newest first, then the one you want
safekeep snapshots list              # size, files, sources, machine, label
safekeep snapshots show <snapshot>   # its sources, sizes and tags
safekeep snapshots show <snapshot> --source ~/.ssh    # one source's files
safekeep files list --from <snapshot>                 # every file it holds
                                     # <snapshot> is the full name, as in
                                     # 2026-08-04T17-32-08

# 2. SAVE WHERE YOU ARE — the state you are about to replace gets a snapshot
safekeep backup run --label 'before rolling back to <snapshot>'

# 3. REHEARSE — names every file the restore would write, changes nothing
safekeep restore -n --to / --from <snapshot> --all --on-conflict overwrite
                                     # + a file this machine lacks
                                     # ~ a file it would replace

# 4. ROLL BACK — the snapshot wins, compared by checksum
safekeep restore --to / --from <snapshot> --all --on-conflict overwrite
safekeep restore --to / --from <snapshot> --source ~/.ssh --on-conflict overwrite
safekeep restore --to / --on-conflict overwrite   # or pick in fzf: a snapshot,
                                     # then sources. tab marks one, ctrl-a
                                     # marks all, enter restores

# 5. CARRY FORWARD — the rolled-back state becomes the newest snapshot
safekeep backup run --label 'rolled back to <snapshot>'
safekeep snapshots list              # it heads the list, with that label.
                                     # Unchanged files hard-link into step 2's
                                     # run, so it costs only what changed

# THE WHOLE THING IN FOUR LINES
safekeep snapshots list
safekeep backup run --label 'before rolling back'
safekeep restore --to / --from <snapshot> --all --on-conflict overwrite
safekeep backup run --label 'rolled back to <snapshot>'
```

## Gotchas

```bash
# - A restore never deletes. A file made after <snapshot> stays on disk, and
#   step 5 backs it up. Before step 5, list what is here that <snapshot> lacks:
comm -13 <(safekeep files list --from <snapshot> --json | jq -r '.[].path' | sort) \
         <(safekeep files list --json | jq -r '.[] | select(.here) | .path' | sort)
# - The default --on-conflict backup keeps <name>.pre-restore beside each file
#   it replaced. No skip pattern matches that name, so step 5 backs every one
#   up. Step 2 already holds those versions, so overwrite loses nothing.
# - --from <date> is that day's LAST run. A rollback to an earlier run the
#   same day needs its full name.
# - A narrowed snapshot (marked in snapshots list) holds only what its run
#   selected. Rolling back to it leaves every other source as it is now.
# - --all writes THROUGH a symlinked dir (~/.config/nvim) into the repo it
#   points at. --skip-symlinked leaves those paths alone.
```
