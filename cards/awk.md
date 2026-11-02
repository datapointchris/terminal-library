---
tags: [awk, text, data, shell]
---

# awk — one-liner forms, and when jq, duckdb or the shell wins

```bash
# FIELDS — the 90% case. Splits on runs of whitespace, which cut cannot do.
awk '{print $2}'                    # second field
awk '{print $NF}'                   # last field; $(NF-1) is second to last
awk '{print $1, $3}'                # OFS between them, a space by default
awk -F: '{print $1}'                # custom separator
awk -F'\t' '{print $3}'             # tab-separated

# LINES — a condition with no block prints the whole line
awk 'NR > 1'                        # drop the header row
awk 'NF'                            # drop blank lines
awk '$3 > 100'                      # numeric test on a field
awk '/fail/'                        # grep, but composable with the above
awk '/fail/ {print $1}'             # match, then extract

# AGGREGATE — BEGIN/END, still one line
awk '{sum += $3} END {print sum}'   # sum a column
awk 'END {print NR}'                # count lines
awk '{s += $3} END {print s/NR}'    # mean
awk '!seen[$1]++'                   # dedupe on field 1, input order kept

# FLOAT MATH — why awk survives in shell scripts at all
awk 'BEGIN {printf "%.2f\n", 22/7}'                # POSIX shell is integer-only
awk -v a="$start" -v b="$end" 'BEGIN {print b - a}'

# SHELL VALUES — always -v, never string interpolation into the program
awk -v want="$name" '$1 == want {print $2}'
```

## Reach for something else

```bash
# JSON — awk cannot see nested structure at all
jq -r '.[] | select(.seconds > 40) | .service'

# GROUP BY, window functions, joins — SQL says in one line what awk says in twenty
duckdb -c "SELECT service, sum(seconds) FROM 'deploys.csv' GROUP BY service"

# One field per line — the shell splits first-word-and-rest natively
while read -r key rest; do :; done   # $key is field 1, $rest is everything after

# cut splits on ONE delimiter char, so column padding becomes empty fields:
cut -d' ' -f2                        # -> blank on any space-aligned table
awk '{print $2}'                     # -> the field; awk splits on runs of whitespace
```

## Which tool

```text
one line, columnar text            ->  awk
input is JSON                      ->  jq
group by / window function / join  ->  duckdb
needs a second line, state, branches -> python
```

The split is one-liner versus program. Accumulator arrays, `NR`-indexed storage and a
`BEGIN`/`END` pair holding state across the stream are where awk becomes write-only.
When a task needs that, it has outgrown awk.
