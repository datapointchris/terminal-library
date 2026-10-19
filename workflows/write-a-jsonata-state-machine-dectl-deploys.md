---
tags: [dectl, aws, step-functions, sfn, jsonata, asl, terraform, templatefile, lambda, glue, yaml, config, variables, escape, recipe]
---

# write a JSONata state machine dectl deploys (asl.json template to config to deploy)

```bash
# GOAL: one .asl.json that Terraform's templatefile() and `dectl sfn deploy`
# render to the same bytes, filled from dectl config written once with variables.
# Below: a Lambda task, a Choice, a Glue job run to completion, and a Pass.
# Rendered for dev it passes ValidateStateMachineDefinition, and Terraform's
# render is byte-identical to dectl's.
```

## 1. THE TEMPLATE — modules/step_functions/process_orders.asl.json

```json
{
  "QueryLanguage": "JSONata",
  "StartAt": "Process",
  "States": {
    "Process": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Arguments": {
        "FunctionName": "${process_function_arn}",
        "Payload": {
          "bucket": "${raw_bucket}",
          "key": "{% 'orders/' & $states.input.date & '.json' %}"
        }
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
        "JobName": "${copy_job_name}",
        "Arguments": { "--source_key": "{% $states.input.key %}" }
      },
      "TimeoutSeconds": ${copy_timeout_seconds},
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
"${name}"           a value inside a JSON string
${name}             outside quotes takes a number or boolean, written as Terraform does
{% ${name} %}       a value inside a JSONata expression: true, 1, a quoted string
$${   %%{           a literal ${ or %{. JSONata's $orders.${ ... } (one object per
                    order) needs it; $orders{ ... } (one grouped object) does not
${f(x)}  %{ if }    refused by dectl rather than approximated: only a bare ${name}
```

## 2. THE CONFIG — dectl config edit

```yaml
variables:                       # top level: reaches every string in the file
  account: "123456789012"        # quoted: unquoted 0123 is octal to YAML
  region: us-east-2
  company: acme
  lambda_arn_prefix: arn:aws:lambda:{region}:{account}:function   # built from the two above
defaults:
  account_id: "{account}"
  region: "{region}"
  environment: dev
pipelines:
  sales:
    resolve_paths_from: ~/code/sales-pipeline
    variables:                   # this pipeline only; cannot redefine a top-level one
      raw_bucket: "{company}-{env}-raw"
    lambdas:
      process:
        name: "{company}-{env}-process-orders"
        source_dir: modules/lambda/process_orders/code
    glue_jobs:
      load:
        name: "{company}-{env}-load-orders"
        script_bucket: "{company}-{env}-scripts"
        scripts: [load_orders.py]
        role: arn:aws:iam::{account}:role/{company}-{env}-glue      # starts with a letter: no quotes
    step_functions:
      process-orders:
        name: "{company}-{env}-process-orders"
        definition_file: modules/step_functions/process_orders.asl.json
        definition_substitutions:            # one per ${name} in the file
          process_function_arn: "{lambda_arn_prefix}:{company}-{env}-process-orders"
          raw_bucket: "{raw_bucket}"
          copy_job_name: "{company}-{env}-load-orders"
          copy_timeout_seconds: 3600         # renders 3600; 2.0 would render 2
          min_orders: 1
          load_enabled: true                 # renders true, never Python's True
    buckets:
      raw: "{raw_bucket}"
```

## 3. CHECK AND SHIP — then the loop in `iterate-on-an-aws-pipeline-with-dectl`

```bash
dectl config validate                         # a bad variable names its key here
dectl sales list                              # every name with the variables filled in
dectl sales sfn process-orders plan           # renders, diffs against live, writes nothing
dectl sales sfn process-orders deploy         # validates with Step Functions, then writes
dectl sales sfn process-orders run --payload-file in.json --follow
```

## Gotchas

```bash
# - A YAML value STARTING with { needs quotes: `x: {env}` is a mapping and
#   `x: {company}-raw` does not parse. A { later in the value is plain text.
# - {name} in config is dectl's variable; ${name} in the file is Terraform's.
#   ${NAME} in config is never a variable. {env} never goes in the file: a
#   Terraform ${env} takes an entry `env: "{env}"`.
# - A {typo} naming no variable fails the load, so it never reaches AWS.
# - The template is not JSON until rendered: an editor flags the bare
#   ${copy_timeout_seconds}. Step Functions rejects a bare top-level $ in an
#   expression; use $states.input, $states.result or an Assign'd $variable.
```
