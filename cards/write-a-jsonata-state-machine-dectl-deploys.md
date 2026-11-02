---
tags: [dectl, aws, step-functions, sfn, jsonata, asl, terraform, templatefile, tftpl, lambda, glue, yaml, anchors, merge, config, variables, escape, recipe]
---

# write a JSONata state machine dectl deploys (asl template to config to deploy)

```bash
# GOAL: one definition template that Terraform's templatefile() and `dectl sfn
# deploy` render to the same bytes, filled from config written once: variables
# for what differs, YAML anchors for what repeats, JSONata in both files.
# Both files below are dectl's tests/layered_pipelines_config, verbatim apart
# from comments. dectl's suite loads the config, resolves both pipelines
# through the installed CLI, and deploys the definition to live Step Functions.
# dectl's README carries the same config with what every value resolves to
# and the exact message for each common mistake.
```

## 1. THE TEMPLATE — flow.asl.json.tftpl

```json
{
  "Comment": "Loads a day's orders when there are enough, then totals them by customer",
  "QueryLanguage": "JSONata",
  "StartAt": "Process",
  "States": {
    "Process": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Arguments": {
        "FunctionName": "${process_arn}",
        "Payload": { "key": "${landing_key}" }
      },
      "Assign": { "orders": "{% $states.result.Payload.orders %}" },
      "Output": "{% $states.result.Payload %}",
      "Next": "HasOrders"
    },
    "HasOrders": {
      "Type": "Choice",
      "Choices": [
        { "Condition": "{% ${load_enabled} and $count($orders) >= ${min_orders} %}", "Next": "Load" }
      ],
      "Default": "NothingToLoad"
    },
    "Load": {
      "Type": "Task",
      "Resource": "arn:aws:states:::glue:startJobRun.sync",
      "Arguments": {
        "JobName": "${load_job}",
        "Arguments": { "--source_key": "{% $states.input.key %}" }
      },
      "TimeoutSeconds": ${load_timeout_seconds},
      "Output": "${loaded_output}",
      "Next": "TotalByCustomer"
    },
    "TotalByCustomer": {
      "Type": "Pass",
      "Output": "{% $orders.$${ customer: $sum(total) } %}",
      "End": true
    },
    "NothingToLoad": { "Type": "Succeed" }
  }
}
```

```text
{% ... %}           JSONata. Needs nothing: only ${ and %{ mean anything to the renderer
"${name}"           a value inside a JSON string, which can itself be a whole {% ... %}
${name}             outside quotes takes a number or boolean, written as Terraform does
{% ${name} %}       a value inside a JSONata expression: true, 1, a quoted string
$${   %%{           a literal ${ or %{. JSONata's $orders.${ ... } (one object per
                    order) needs it; $orders{ ... } (one grouped object) does not
${f(x)}  %{ if }    refused by dectl rather than approximated: only a bare ${name}
```

## 2. THE CONFIG — dectl config edit

```yaml
variables:                       # top level: reaches every string in the file
  company: acme
  account: "123456789012"        # quoted: unquoted 0123 is octal to YAML
  region: us-east-2
  lambda_arns: arn:aws:lambda:{region}:{account}:function   # built from the ones above it
  glue_role: arn:aws:iam::{account}:role/{company}-{env}-glue

defaults:                        # settings dectl reads itself; variables only fill text
  account_id: "{account}"
  region: "{region}"
  environment: dev

jenkins:
  url: https://jenkins.example.com
  user: ${JENKINS_USER}          # ${NAME} is never a variable: read from the environment
  token: ${JENKINS_API_TOKEN}

pipelines:
  sales: &pipeline               # anchor the whole pipeline so finance can copy it
    # resolve_paths_from: ~/code/orders-pipeline   # set it here and the merge gives it to finance
    variables:                   # this pipeline only; cannot redefine a top-level one
      pipeline: sales
      dataset: orders
      prefix: &prefix "{company}-{env}-{pipeline}"    # anchored so finance repeats it whole
      raw_bucket: &raw-bucket "{prefix}-raw"          # used INSIDE other values: a variable
      landing: &landing s3://{raw_bucket}/{dataset}/incoming/
    buckets:
      raw: "{raw_bucket}"
      curated: &curated-bucket "{prefix}-curated"     # used only WHOLE: an anchor is enough
    lambdas:
      process:
        name: "{prefix}-process-{dataset}"
        source_dir: lambdas/process
        live_alias: live                   # deploy --publish repoints it
      workflow:
        name: "{prefix}-workflow"
        source_dir: lambdas/workflow
        live_alias: live
        durable: true                      # run starts an execution; adds executions and history
    glue_jobs:
      load: &load
        name: "{prefix}-load"
        script_bucket: "{prefix}-scripts"
        scripts: [load.py]
        role: "{glue_role}"
        max_capacity: 1
        arguments:
          SOURCE: s3://{raw_bucket}/{dataset}/          # starts with a letter: no quotes
          LANDING: "{landing}"
          TARGET_BUCKET: *curated-bucket                # whole and unquoted, never "{curated-bucket}"
          OUTPUT_PATTERN: out/{{date}}/                 # reaches Glue as out/{date}/
      backfill:
        <<: *load                # every key of load; the keys below replace theirs
        name: "{prefix}-backfill"
        scripts: [backfill.py]
    step_functions:
      flow:
        name: "{prefix}-flow"
        log_group: /aws/vendedlogs/states/{prefix}-flow   # what monitor tails
        definition_file: flow.asl.json.tftpl
        live_alias: live
        definition_substitutions:            # one per ${name} in the file
          process_arn: "{lambda_arns}:{prefix}-process-{dataset}"
          load_job: "{prefix}-load"
          load_timeout_seconds: 3600         # renders 3600; 2.0 would render 2
          min_orders: 1
          load_enabled: true                 # renders true, never Python's True
          landing_key: "{% '{landing}' & $states.input.date & '.json' %}"     # whole JSONata
          loaded_output: "{% $merge([$states.input, {'stage': 'loaded'}]) %}"
    iceberg_tables:
      events:
        database: "{company}_{env}_{pipeline}"
        table: "{dataset}"
    monitor:
      lambdas: [process, workflow]
      step_functions: [flow]
    jenkins:
      job_path: data-engineering/{pipeline}-deploy
      parameters:
        ENVIRONMENT: "{env}"
      actions:                   # each key is a verb: dectl sales jenkins apply
        apply:
          ACTION: deploy
        plan:
          ACTION: dry_run

  finance:
    <<: *pipeline                # sales' blocks, filled with finance's variables
    variables:                   # replaces sales' block whole: restate every variable
      pipeline: finance
      dataset: invoices
      prefix: *prefix            # the same text, so it resolves to finance's pipeline
      raw_bucket: *raw-bucket
      landing: *landing
```

Under `--env prod`, finance resolves to `acme-prod-finance-raw`, `acme-prod-finance-backfill`,
`acme-prod-finance-flow`, a `TARGET_BUCKET` of `acme-prod-finance-curated`, and a `landing_key` of
`{% 's3://acme-prod-finance-raw/invoices/incoming/' & $states.input.date & '.json' %}`.

## 3. CHECK AND SHIP — then the loop in `iterate-on-an-aws-pipeline-with-dectl`

```bash
dectl config validate                         # a bad variable names its key and its fix
dectl --env prod finance list                 # every name with variables and the env filled in
dectl finance sfn flow plan                   # renders, validates, diffs against live, writes nothing
dectl finance sfn flow deploy                 # the same, then writes
dectl finance sfn flow run --payload-file in.json --follow
```

## Gotchas

```bash
# - A YAML value STARTING with { needs quotes: `x: {env}` is a mapping and
#   `x: {company}-raw` does not parse. A { later in the value is plain text.
#   With an anchor, the quotes go after it: `raw: &raw "{prefix}-raw"`.
# - {name} in config is dectl's variable; ${name} in the file is Terraform's.
#   ${NAME} in config is never a variable. {env} never goes in the file: a
#   Terraform ${env} takes an entry `env: "{env}"`.
# - A {typo} naming no variable fails the load, so it never reaches AWS. A
#   literal {date} for a Glue job or Jenkinsfile to fill is written {{date}}.
#   Never escape a variable you defined: {{x}} ships the braces to AWS.
#   The message offers a close name too: {date} gets "Did you mean {dataset}?"
#   beside the {{date}} escape.
# - A name used INSIDE other values is a variable ({raw_bucket}). One value
#   repeated WHOLE can be an anchor: `TARGET_BUCKET: *curated-bucket`.
#   "{curated-bucket}" looks for a variable and is refused with the line of
#   &curated-bucket and the *curated-bucket that works. "*curated-bucket" in
#   quotes is text and is refused. s3://*curated-bucket/out/ is that literal
#   text: an alias never goes inside a longer value.
# - A variable from another pipeline's block is out of reach. validate names
#   the block, and says to define it in this pipeline too when its value uses
#   pipeline variables, or offers the top level when it does not.
# - A JSONata substitution lands inside a JSON string, so its string literals
#   take single quotes and the YAML value takes double quotes.
# - YAML copies an anchored block before any variable is filled, so the copy
#   resolves against the pipeline it lands in. `<<:` merges one level deep:
#   a `variables:` written in the copy replaces the copied block whole.
# - An anchor is written where its block is first used. The config has no key
#   for holding anchors alone, and a block under `variables:` is refused. A
#   *name above its &name in the file is YAML's "found undefined alias".
# - The template is not JSON until rendered: an editor flags the bare
#   ${load_timeout_seconds}. .tftpl is Terraform's extension for a
#   templatefile() template and keeps JSON linters off it; dectl takes any.
# - Step Functions refuses a bare top-level $ in an expression:
#   "UNSUPPORTED_JSONATA_EXPRESSION ... Reference to '$' at the top level is
#   not supported". Use $states.input, $states.result or an Assign'd $orders.
```
