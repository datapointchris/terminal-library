---
tags: [sparkxray, spark, pyspark, event-log, performance, slow, tuning, skew, spill, shuffle, query-plan, aqe, executors, stages, history-server, glue, aws, s3, cloudwatch, logs, ci, diff, query, sql, duckdb, parquet, debugging, data-engineering, recipe]
---

# find why a Spark run was slow (sparkxray findings to stages to plans to diff)

```bash
# GOAL: take a slow Spark run's event log to its cause, the fix, a report and
# a CI gate, on a laptop, with no JVM and no cluster.
# Shape: sparkxray RESOURCE VERB [LOG] [ARGS]. LOG is a file or directory, an
# s3:// URI, a History Server or driver UI address, or the app id of a log
# already read. The first command reads LOG into the cache; every later one
# opens it by the <app-id> the stderr footer names. Every command's stderr
# ends on the next command to run.

# 0. GET A LOG — Spark writes one only when told to
#    spark.eventLog.enabled=true  spark.eventLog.dir=s3://your-bucket/spark-logs/
#    Glue job: --enable-spark-ui true  --spark-event-logs-path s3://your-bucket/spark-logs/
#    Spark 4 writes a directory, eventlog_v2_<app-id>/; pass the directory
aws s3 ls s3://your-bucket/spark-logs/            # one object or directory per run

# 1. WHAT SLOWED IT — costliest first, by the wall time a fix could recover
sparkxray findings list ./slow.zstd               # exits 3 on a high finding
sparkxray findings list <app-id> --min-severity high          # hides rows only
sparkxray findings list <app-id> --rule stage.shuffle-skew    # lists and gates one
sparkxray findings show <app-id> stage.shuffle-skew:s4        # evidence, the fix
sparkxray rules show stage.shuffle-skew           # what it looks for, its lesson
sparkxray rules list                              # every rule, the signals it reads
#    an id is rule:scope. s4 stage 4, e0 SQL 0, e0n9 node n9 of SQL 0's last
#    plan, x3 executor 3, app the whole run. Severity moves one step by the
#    scope's share of wall time: down under 5%, up over 50%.

# 2. THE STAGE — where the time went, then the partition it waited on
sparkxray stages list <app-id> --sort wall_ms -n 5             # longest stages
sparkxray stages list <app-id> --sort wasted_core_hours -n 3   # most idle cores
sparkxray stages show <app-id> 4                  # min/p25/p50/p75/p95/max a metric
sparkxray tasks list <app-id> --stage 4 --sort duration_ms -n 5   # its slowest
#    max far above p50 is skew; one task holding the shuffle read is the key

# 3. THE QUERY — which SQL execution, and which node of its plan
sparkxray sql list <app-id>                       # each one's share of the run
sparkxray sql show <app-id> 0                     # jobs, operators, what AQE changed
sparkxray plans show <app-id> 0                   # last plan, findings on its nodes
sparkxray plans show <app-id> 0 --initial         # the plan before AQE re-planned
sparkxray plans show <app-id> 0 --plan-version 1  # one AQE step at a time
sparkxray plans show <app-id> 0 --json            # each metric's min/median/max

# 4. EXECUTORS — too many, too few, or idle
sparkxray executors list <app-id>                 # lifetime, idle share, peak heap
sparkxray executors simulate <app-id>             # 10% to 500% of today's count
sparkxray executors simulate <app-id> --executors 2..64
sparkxray executors simulate <app-id> --executors 4,8,16 --cores 4
#    critical_path_ms is the floor. Past it more executors buy only core-hours

# 5. AGAINST A GOOD RUN — the same job, the run that was fine first
sparkxray logs diff ./good.zstd ./slow.zstd       # timing, settings, findings
sparkxray logs diff ./good.zstd ./slow.zstd --fail-on high    # 3: B added one
sparkxray logs diff ./good.zstd ./slow.zstd --baseline ./older-good.zstd \
  --ignore-config 'spark.app.*'                   # its exceptions are routine

# 6. WRITE IT UP — same log, thresholds and version give the same bytes
sparkxray reports write <app-id> --out reports/slow.md          # for the ticket
sparkxray reports write <app-id> --format json --out reports/slow.json

# 7. GATE CI — 0 passed, 2 usage error, 3 a finding at the bar or no log read
sparkxray findings list s3://your-bucket/spark-logs/<app-id>.zstd   # bar: high
sparkxray findings list <app-id> --fail-on critical
sparkxray findings list <app-id> --json | jq -c .gate      # passed, blocking ids
sparkxray findings list <app-id> --json \
  | jq -r '.findings[] | select(.severity == "high") | .finding_id'

# 8. TUNE A RULE — each threshold says who set it: default, flag or file
sparkxray rules show stage.shuffle-skew --threshold stage.shuffle-skew.ratio=8
sparkxray findings list <app-id> --threshold stage.shuffle-skew.min_bytes=512MiB
cat > thresholds.toml <<'EOF'
["stage.shuffle-skew"]
ratio = 8
min_bytes = "512MiB"
EOF
sparkxray findings list <app-id> --thresholds thresholds.toml   # a flag beats it

# 9. YOUR OWN SQL — DuckDB over the log's Parquet tables
sparkxray tables list <app-id>                    # tables, files, columns, units
sparkxray tables query <app-id> "select stage_id, max(duration_ms) from tasks_effective group by all order by 2 desc limit 3"
sparkxray tables query <app-id> "select rule_id, severity, impact_seconds from findings order by impact_seconds desc nulls last"
duckdb -c "select stage_id, task_id, duration_ms from '$(sparkxray logs ingest <app-id>)/tasks.parquet' order by 3 desc limit 5"
sparkxray metrics show "Executor CPU Time"        # a Spark field: unit, source
sparkxray metrics show "spill size"               # a plan metric, as plans shows it
sparkxray metrics list                            # every unit, no log needed

# 10. JOIN WHAT THE LOG LACKS — findings list names each rule it skipped
aws glue get-job-runs --job-name <job-name> \
  --query 'JobRuns[].[Id,StartedOn,JobRunState]' --output table
sparkxray logs ingest <app-id> --glue-run <job-name>:<run-id>   # CloudWatch
sparkxray logs ingest <app-id> --driver-log s3://your-bucket/driver-logs/stderr.log.zstd
sparkxray logs ingest <app-id> --driver-log cloudwatch:<log-group>:<log-stream>
sparkxray findings list <app-id> --rule app.driver-oom --rule app.driver-memory-pressure

# 11. A LOG ELSEWHERE — S3 through boto3's credentials, a server over HTTP
sparkxray findings list s3://your-bucket/spark-logs/<app-id>.zstd   # one object
sparkxray findings list http://history-server:18080/history/<app-id>/
sparkxray logs ingest http://history-server:18080 --app <app-id>    # one of many

# 12. THE CACHE — $XDG_CACHE_HOME/sparkxray/, never beside your log
sparkxray logs list                               # each log read, by its app id
sparkxray logs show <app-id>                      # its files, and what it lacks
sparkxray logs ingest ./slow.zstd                 # read ahead, print tables' dir
rm -rf "${XDG_CACHE_HOME:-$HOME/.cache}/sparkxray"   # empty it

# 13. FOR A PROGRAM OR AN AGENT — data on stdout, everything else on stderr
sparkxray capabilities show --json | jq -r '.commands[].path | join(" ")'
sparkxray schemas list                            # each document, its command
sparkxray schemas show findings                   # JSON Schema, draft 2020-12
sparkxray rules show stage.shufle-skew --json 2>&1 >/dev/null | jq .error
#    {"code": "E-RULE-001", "message": "...; did you mean stage.shuffle-skew?",
#     "field": "rule_id", "valid": [...], "hint": "sparkxray rules list"}
#    a hint starting sparkxray runs as written

# THE WHOLE THING IN FOUR LINES
sparkxray findings list ./slow.zstd               # what, costliest first
sparkxray findings show <app-id> <finding-id>     # why, and the fix
sparkxray plans show <app-id> <sql-id>            # where in the query
sparkxray logs diff ./good.zstd ./slow.zstd       # what changed since it was fine
```

## Gotchas

```bash
# - Exit 3 means a finding at or above --fail-on (high by default), or a log
#   that could not be read. Under --json, exit 3 with nothing on stdout means
#   the log was not read. A mistyped app id is exit 3, with a did-you-mean.
# - --min-severity and -n hide rows and never move the gate. --rule moves it.
# - A raised threshold can remove a finding and its id. With
#   --threshold stage.shuffle-skew.ratio=80, findings show <app-id>
#   stage.shuffle-skew:s4 refuses with E-QUERY-002 and offers
#   stage.spill-skew:s4, the rule that fires in its place.
# - tables query's findings table is the last analysis run on the log,
#   overrides included. Run findings list at the thresholds you mean first.
# - An S3 prefix, or a History Server holding several runs, refuses with
#   E-INGEST-005 and lists each. Pass one object, --app <app-id>, or the
#   /history/<app-id>/ address.
# - Stage and SQL ids shift between runs. logs diff pairs by plan and stage
#   name, never id, and prints how: "SQL 1 → 0, paired by initial_plan".
# - A --baseline that failed marks no exception routine. stderr says so.
# - executors simulate replays each task at the time it took. It cannot see
#   what changes task times: memory easing spill or GC, or AQE splitting skew.
# - A live driver UI serves no event log, so sparkxray rebuilds one from its
#   REST JSON. That lacks stage parents, accumulator updates, executor peaks
#   and changed settings. logs show names what is missing.
# - --driver-log reads only Spark 4's structured log, written with
#   spark.log.structuredLogging.enabled=true. Plain log4j is E-ENRICH-002.
# - --glue-run reads only a run with --enable-observability-metrics true.
#   CloudWatch lists no metric silent for two weeks, so an older run joins
#   none and warns E-ENRICH-003. Region: AWS_DEFAULT_REGION or the profile.
# - An upgrade of sparkxray rebuilds the tables without the joined ones. Run
#   logs ingest with --glue-run and --driver-log again after one.
# - metrics show knows table columns, plan metrics and Spark fields. A
#   summary's own number, as wall_ms in stages list, is on that --help.
```
