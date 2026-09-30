# path-filter.yml

Detects which parts of the repository changed, so the following jobs run only for those, or not at all. It avoids
running the backend tests for a change to the documentation, and keeps a monorepo pipeline fast.

```yaml
paths:
  uses: Mitchou10/github-workflow/.github/workflows/path-filter.yml@v0
  permissions:
    contents: read
    pull-requests: read
  with:
    FILTERS: |
      backend:
        - 'backend/**'
      frontend:
        - 'frontend/**'

test-backend:
  needs: paths
  if: contains(fromJSON(needs.paths.outputs.changed), 'backend')
  uses: ./.github/workflows/test-backend.yml
```

A full pipeline with a status gate is in [examples/path-filter/caller.yml](../examples/path-filter/caller.yml).

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `FILTERS` | required | YAML mapping of filter name to a list of glob patterns |
| `ALWAYS` | empty | Comma-separated filter names that make everything count as changed |
| `ALL_ON_EVENTS` | `workflow_dispatch,schedule,release` | Events for which everything counts as changed |
| `SKIP_DRAFT` | `true` | Report nothing changed on draft pull requests |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

## Outputs

| Output | Description |
| --- | --- |
| `changed` | JSON array of the filters that changed, for example `["backend","helm"]` |
| `any` | `true` when at least one filter changed |

Use them in the `if` of the next jobs:

```yaml
if: contains(fromJSON(needs.paths.outputs.changed), 'backend')   # one filter
if: needs.paths.outputs.any == 'true'                             # anything at all
if: contains(fromJSON(needs.paths.outputs.changed), 'backend') || contains(fromJSON(needs.paths.outputs.changed), 'ci')
```

A single output holding the list is deliberate. Outputs of a reusable workflow are declared in advance, so one
output per filter would limit the workflow to filters it already knows about.

## Writing the filters

The syntax is the one of [dorny/paths-filter](https://github.com/dorny/paths-filter). Patterns are globs relative
to the repository root:

```yaml
FILTERS: |
  backend:
    - 'backend/**'
    - '!backend/docs/**'          # exclude
  helm:
    - 'helm/**'
    - 'ci/configs/ct.yaml'        # a file outside the folder that also matters
  python-deps:
    - '**/uv.lock'
```

Quote the patterns: an unquoted `*` starts a YAML alias. Use simple names (letters, digits, `-`, `_`) as they
end up in a JSON array and in `contains(...)`.

## Files that should run everything

Some changes affect every part: the CI workflows themselves, a shared lockfile, a root config. Give them a filter
and list it in `ALWAYS`:

```yaml
FILTERS: |
  backend:
    - 'backend/**'
  ci:
    - '.github/workflows/**'
ALWAYS: ci
```

When `ci` changes, `changed` contains **all** the filters, so every job runs.

## When there is nothing to compare with

| Situation | `changed` |
| --- | --- |
| Pull request | Files of the pull request, against its base |
| Push to a branch | Files between the previous and the new commit |
| `workflow_dispatch`, `schedule`, `release` (see `ALL_ON_EVENTS`) | All the filters |
| Tag push | All the filters |
| Draft pull request, with `SKIP_DRAFT` | Nothing (`[]`) |

Running everything when in doubt is the safe choice: a skipped test is worse than a useless one. Remove an event
from `ALL_ON_EVENTS` if you do not want that.

## With the status gate

A job skipped because its folder did not change is `skipped`, not `failed`. The [status gate](status-gate.md)
accepts skipped jobs by default (`ALLOW_SKIPPED: true`), so it stays green and can be the only required check.
Put `paths` in the gate's `needs` too: if the detection itself fails, the gate fails.

## Permissions

The workflow declares none: it gets those of the calling job. It needs `contents: read`, and `pull-requests: read`
because the files of a pull request are read through the API.

## Limits

- On a very large pull request (more than 3000 files), the API lists a truncated set of files; the action falls
  back on git in that case, which works because the repository is checked out.
- A pull request that only changes a file matched by no filter reports `changed: []`, and all the guarded jobs
  are skipped. Add a filter (or `ALWAYS`) for files that should trigger something.
- Squashed or force-pushed history on a push event can make the comparison base unknown; the action then
  compares with the default branch.
