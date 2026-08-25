---
tags: [git, github, rebase, merge, conflicts, pull-requests]
cadence: 2mo
---

# Diagnose a branch that will not merge

> A PR refuses to merge and `gh` suggests `--auto`, which does not merge — it
> queues one for later. This Lab builds a branch that is both conflicting and
> diverged, then drills the three questions that tell them apart. Everything
> runs on a scratch repo with a real remote, so `push` behaves for real.

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

4. **Rebase and resolve.** `git rebase main`, edit `notes.md` to keep both
   lines, `git add notes.md`, `git rebase --continue`
   - Expect: "Successfully rebased".
   - Why: rebase replays your commits onto the new base. Resolving here is the
     same work as a merge, done once per commit instead of once at the end.

5. **Read the signal that means force-push.** `git status`
   - Expect: `[ahead 2, behind 1]` and *"Your branch and 'origin/feature' have
     diverged"*.
   - Why: counts on **both** sides means the rebase rewrote commits the remote
     still has. A plain `push` is rejected; the remote is not behind, it is
     holding different history. This is the state where re-rebasing changes
     nothing, because the remote never saw the first one.

6. **Push the rewrite safely.**
   `git push --force-with-lease --force-if-includes`
   - Expect: `+ <old>...<new> feature -> feature (forced update)`.
   - Why: `--force-with-lease` refuses if origin moved since your fetch,
     `--force-if-includes` refuses if your ref lacks what you last fetched.
     A bare `--force` skips both and can erase someone else's push.

7. **Confirm from GitHub's side**, on a real PR rather than the scratch repo.
   `gh pr view <n> --json mergeable,mergeStateStatus`
   - Expect: `MERGEABLE` / `CLEAN`. `UNKNOWN` means GitHub is still computing
     after a push — wait and ask again.

## The whole thing in one breath

```bash
gh pr view <n> --json mergeable,mergeStateStatus   # CONFLICTING? then:
git fetch origin && git rebase origin/main         # resolve, --continue
git push --force-with-lease --force-if-includes
```
