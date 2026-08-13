---
tags: [awk, text, data, shell]
cadence: 1mo
---

# Read fields and aggregate with awk

> awk earns its place as a one-liner over columnar text. This Lab drills the forms
> you actually meet, and ends by showing where jq and the shell do it better.

## Setup

```bash
LAB=$(mktemp -d) && cd "$LAB"
cat > deploys.txt <<'EOF'
service env  seconds status
api     prod 42      ok
api     dev  17      ok
web     prod 93      fail
web     prod 31      ok
worker  dev  58      fail
EOF
```

## Steps

1. **Print a field.** `awk '{print $1}' deploys.txt`
   - Expect: `service api api web web worker`, one per line.
   - Why: `$1` is the first whitespace-separated field, `$0` the whole line. Fields
     split on *runs* of whitespace, so the column padding does not create empties.

2. **Drop the header.** `awk 'NR > 1 {print $1}' deploys.txt`
   - Expect: the same without `service`.
   - Why: `NR` is the current line number. A condition before `{}` gates the block.

3. **Last field.** `awk 'NR > 1 {print $NF}' deploys.txt`
   - Expect: `ok ok fail ok fail`.
   - Why: `NF` is the field count, so `$NF` is the last one. `$(NF-1)` is the one before.

4. **Test a field numerically.** `awk 'NR > 1 && $3 > 40 {print $1, $3}' deploys.txt`
   - Expect: `api 42`, `web 93`, `worker 58`.
   - Why: awk types a field by use, so `$3 > 40` compares numbers. The comma in
     `print` inserts OFS, a space.

5. **Match, then extract.** `awk '/fail/ {print $1}' deploys.txt`
   - Expect: `web`, `worker`.
   - Why: a bare `/regex/` is a condition against the whole line. This is grep and
     cut in one pass.

6. **Sum a column.** `awk 'NR > 1 {sum += $3} END {print sum}' deploys.txt`
   - Expect: `241`.
   - Why: `END` runs once after the last line. Variables need no declaration and
     start at zero.

7. **Deduplicate, keeping order.** `awk 'NR > 1 && !seen[$1]++ {print $1}' deploys.txt`
   - Expect: `api`, `web`, `worker`.
   - Why: `seen[$1]++` returns the old count, so it is 0 (false) only the first
     time. `!` makes that first sighting the one that prints. This is the idiom
     `sort -u` cannot do, because it would reorder.

8. **Float arithmetic.** `awk 'BEGIN {printf "%.2f\n", 241/5}'`
   - Expect: `48.20`.
   - Why: `BEGIN` runs before any input, so no file is needed. POSIX shell has
     integer math only, which is why this form shows up inside shell scripts.

9. **Pass a shell value in.** `awk -v want=prod 'NR > 1 && $2 == want {print $1}' deploys.txt`
   - Expect: `api`, `web`, `web`.
   - Why: `-v` binds a variable before the program runs. Interpolating `"$name"`
     into the program text instead lets the value change the program.

## Where something else wins

```bash
tail -n +2 deploys.txt | cut -d' ' -f2   # five blank lines
awk 'NR > 1 {print $2}' deploys.txt      # prod dev prod prod dev
                              # cut splits on ONE space, so the column padding
                              # becomes empty fields. awk splits on runs.

tail -n +2 deploys.txt | while read -r svc env secs state; do echo "$svc=$secs"; done
                              # the shell splits first-word-and-rest with no awk,
                              # but it has no NR, so dropping the header costs a pipe
                              # name it `state`, not `status` — zsh reserves that
```

## The whole thing in one breath

```bash
awk 'NR > 1 && $2 == "prod" {s += $3} END {printf "%.1fs in prod\n", s}' deploys.txt
awk 'NR > 1 {print $NF}' deploys.txt | sort | uniq -c    # status tally
```
