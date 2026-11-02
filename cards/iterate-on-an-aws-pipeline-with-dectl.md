---
tags: [dectl, aws, pipeline, glue, lambda, step-functions, sfn, s3, iceberg, jenkins, terraform, deploy, logs, data-engineering, recipe]
---

# iterate on an AWS pipeline (dectl plan to deploy to run to logs to jenkins)

```bash
# GOAL: change a pipeline's code and watch it run on AWS, then ship it through
# Jenkins. Terraform creates the resources; dectl writes only the code.
# Shape: dectl PIPELINE RESOURCE ALIAS VERB. The verb is last, so the loop on
# one resource changes only the final word. <job>, <fn>, <machine>, <bucket>
# and <table> are the config's aliases, never the AWS names.

# 0. ORIENT — none of these reach AWS except search
dectl env                                  # which {env} is active, and what set it
dectl <pipeline> list                      # each alias beside its real AWS name
dectl config validate                      # a missing command means a bad config
dectl reference                            # the whole grammar, whatever the config
dectl search <keyword>                     # an AWS name the config does not hold

# 1. PLAN — a write's rehearsal is a plan verb beside it, never a flag
dectl <pipeline> glue <job> plan           # definition diff; uploads nothing
dectl <pipeline> sfn <machine> plan        # rendered .asl.json against the live one

# 2. DEPLOY — to what you test with, not what triggers run
dectl <pipeline> glue <job> deploy         # upload scripts, confirm the diff (--yes)
dectl <pipeline> lambda <fn> deploy        # zip source_dir, update $LATEST only
dectl <pipeline> sfn <machine> deploy      # writes only when the bytes changed

# 3. RUN AND WATCH
dectl <pipeline> glue <job> run --follow   # exits non-zero when the run failed
dectl <pipeline> lambda <fn> run --payload-file event.json
echo '{"key": 1}' | dectl <pipeline> lambda <fn> run --payload-file -
dectl <pipeline> sfn <machine> run --payload-file in.json --follow
dectl <pipeline> monitor                   # every monitored log group, one stream
eval "$(dectl <pipeline> s3 export)"       # $<pipeline>_<bucket>, each - as _
aws s3 ls "$(dectl <pipeline> s3 <bucket> uri)"

# 4. WHY DID IT FAIL
dectl <pipeline> glue <job> runs           # recent runs; --json to script them
dectl <pipeline> glue <job> logs --follow  # output and error logs, latest run
dectl <pipeline> sfn <machine> logs        # the latest execution's state history
dectl <pipeline> lambda <fn> executions --status FAILED   # durable: true only
dectl <pipeline> lambda <fn> history <name-tail>          # its steps and retries

# 5. PUBLISH — only now does anything invoking through the alias see it
dectl <pipeline> lambda <fn> deploy --publish             # version, move live_alias
dectl <pipeline> sfn <machine> deploy --publish

# 6. SHIP — the Jenkins job stays the canonical deploy
dectl <pipeline> jenkins                   # the actions the config gives the job
dectl <pipeline> jenkins <action> --follow # queue a build, stream its console
dectl <pipeline> jenkins status            # the last build's result

# 7. A NUMBER MOVED — Iceberg history, read from metadata, no query engine
dectl <pipeline> iceberg <table> diff <snapshot>   # every commit since, and deltas

# THE LOOP IN THREE LINES, then prod is one flag
dectl <pipeline> glue <job> plan && dectl <pipeline> glue <job> deploy
dectl <pipeline> glue <job> run --follow   # edit, then from the top
dectl --env prod <pipeline> jenkins <action> --follow
```

## Gotchas

```bash
# - Every command has a one-token shortcut, shown in parentheses in --help.
#   In the starter config, `dectl exgsr` is
#   `dectl example-pipeline glue source-copy run`. It is ONE
#   token: `dectl ex g sc` fails. The sequences come from your own config.
# - A name with no {env} ignores --env. `--env prod` against my-dev-fn deploys
#   dev and succeeds; dectl warns once on stderr.
# - sfn plan and deploy exist only on a machine whose config names a
#   definition_file. A deploy left uncommitted is undone by the next
#   `terraform apply`, so commit the .asl.json you deployed from. Writing
#   that template, JSONata and $${ included: `write-a-jsonata-state-machine-dectl-deploys`.
# - Deploy paths resolve from the working directory unless the pipeline sets
#   resolve_paths_from. `dectl <pipeline> list` marks which.
# - A durable execution over 15 minutes needs `run --async --name <name>`. The
#   name is the handle history and logs take, by any unique tail.
```
