---
tags: [git, github, merge, conflicts, pull-requests]
---

# Diagnose a branch that will not merge

> A PR refuses to merge and `gh` suggests `--auto`, which does not merge — it
> queues one for later. This Lab builds a branch that is both conflicting and
> diverged, drills the three questions that tell them apart, then catches it
> up by merging, with no force push. Everything runs on a scratch repo with a
> real remote, so `push` behaves for real.

## Setup

```bash
LAB=$(mktemp -d) && cd "$LAB"
git init -q --bare origin.git
git clone -q origin.git work && cd work
git config user.email t@t && git config user.name t
printf 'alpha\nbeta\n' > notes.md
git add notes.md && git commit -qm init && git push -q origin HEAD:main
git branch -qM main && git switch -qc feature
printf 'alpha\nbeta\nFROM FEATURE\n' > notes.md
git add notes.md && git commit -qm "feature line" && git push -q -u origin feature
git switch -q main                          # someone else lands on main
printf 'alpha\nbeta\nFROM MAIN\n' > notes.md
git add notes.md && git commit -qm "main line" && git push -q origin main
git switch -q feature
```

## Steps

1. **Ask what is different, both ways.** `git log --oneline main..HEAD` then
   `git log --oneline HEAD..main`
   - Expect: one commit each way.
   - Why: `A..B` is "commits in B that are not in A" and it is the whole
     vocabulary. Both directions is the complete picture of any two refs.

2. **Find where they last agreed.** `git merge-base main HEAD`
   - Expect: the `init` commit's sha.
   - Why: `git log <that>..main` then shows exactly what arrived under you,
     which is what explains why a branch broke *now*.

3. **Merge in memory, touch nothing.**
   `git merge-tree --write-tree --name-only main HEAD`
   - Expect: `notes.md` listed, and `CONFLICT (content)`.
   - Why: it names the conflicting paths before you start, so you know whether
     you are facing one file or forty.

4. **Merge the base in and resolve.**
   `git merge main -m "chore: merge main into feature"`, edit `notes.md` to
   keep both lines, `git add notes.md`, `git merge --continue`, save the
   message as it is
   - Expect: `CONFLICT (content): Merge conflict in notes.md`, then a commit
     named `chore: merge main into feature`.
   - Why: a merge meets every conflict once, at the end, and every commit the
     branch already holds keeps its sha. A rebase replays each commit as a new
     sha, which is why it ends in a force push.

5. **Read the signal that means a plain push.** `git status`
   - Expect: *"Your branch is ahead of 'origin/feature' by 2 commits."*
   - Why: counts on **one** side only. The merge added `main line` and the
     merge commit on top of what the remote holds, so the remote is simply
     behind. Until you push, GitHub still reads the old head, and merging
     again changes nothing.

6. **Push without force.** `git push`
   - Expect: `<old>..<new> feature -> feature`, with no `(forced update)`.
   - Why: the new head descends from the old one, so the push fast-forwards.

7. **Let another box push to the branch, then commit here.**

   ```bash
   git clone -q -b feature ../origin.git ../other
   git -C ../other -c user.name=t -c user.email=t@t commit -q --allow-empty -m "other box"
   git -C ../other push -q
   git commit -q --allow-empty -m "local work" && git push
   ```

   - Expect: `! [rejected] feature -> feature (fetch first)`.
   - Why: the remote holds a commit you lack, and a plain push only
     fast-forwards. Git refuses rather than drop the other box's commit.

8. **Read the signal that means the remote moved.** `git fetch && git status`
   - Expect: *"Your branch and 'origin/feature' have diverged, and have 1 and
     1 different commits each"*.
   - Why: counts on **both** sides means each copy holds a commit the other
     lacks. Neither is wrong, so neither gets overwritten.

9. **Merge the remote branch in, then push.**
   `git merge origin/feature -m "chore: merge origin/feature into feature"`,
   then `git push`
   - Expect: *"Merge made by the 'ort' strategy."*, then a fast-forward push.
   - Why: the merge descends from both heads, so the push fast-forwards again.
     `git pull` makes the same merge, unless `pull.ff = only` is set, where it
     refuses.

10. **Confirm from GitHub's side**, on a real PR rather than the scratch repo.
    `gh pr view <n> --json mergeable,mergeStateStatus`
    - Expect: `MERGEABLE` / `CLEAN`. `UNKNOWN` means GitHub is still computing
      after a push — wait and ask again.

## The whole thing in one breath

```bash
gh pr view <n> --json mergeable,mergeStateStatus   # CONFLICTING? then:
git fetch origin && git merge origin/main -m "chore: merge main into <branch>"
git push                                           # once resolved: git add, git merge --continue
```
