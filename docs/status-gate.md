# status-gate.yml

One job that goes green only when every job it depends on went green. Require **this one check** in the branch
protection, instead of each job.

```yaml
jobs:
  lint:
    uses: Mitchou10/github-workflow/.github/workflows/python-lint.yml@v0
  test:
    uses: ./.github/workflows/tests.yml

  gate:
    if: ${{ always() }}
    needs: [lint, test]
    uses: Mitchou10/github-workflow/.github/workflows/status-gate.yml@v0
    with:
      NEEDS: ${{ toJSON(needs) }}
```

## Why

Listing every job as a required check has two problems: the list drifts when jobs are added or renamed, and
jobs that are skipped (path filters, drafts) never report a status, so they either block the merge forever or
must be left out. The gate is one stable name, and it decides what counts.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `NEEDS` | required | The caller's `needs` context: `${{ toJSON(needs) }}` |
| `ALLOW_SKIPPED` | `true` | Treat a skipped job as a success |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

## Rules

- A job with the result `success` passes. `failure` and `cancelled` fail the gate.
- `skipped` passes unless `ALLOW_SKIPPED: false`. Keep it `true` when jobs are conditional, as in a pipeline
  with path filters.
- Jobs skipped by a [path filter](path-filter.md) are `skipped`: this is why they pass by default.
- An empty `NEEDS` fails: a gate that checks nothing would always be green.

## Two details that matter

1. **`if: ${{ always() }}` on the calling job is mandatory.** Without it the gate is skipped as soon as a job
   fails, and a skipped required check counts as passed: the merge would be allowed.
2. **Every job of the workflow must be in `needs`.** The gate only knows the jobs it is given. Add each new job
   to the list, or its failure no longer blocks anything.

## Required check name

In the branch protection or the ruleset, require the check named `<job id> / Check jobs status`, for example
`gate / Check jobs status`. Renaming the calling job renames the check.

The gate can only check the workflow it is in. If several workflows must pass, put the gate in one workflow that
calls the others, or require the gate of each.
