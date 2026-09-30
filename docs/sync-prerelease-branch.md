# sync-prerelease-branch.yml

Rebases the prerelease branch (`dev`) on the release branch (`main`). See
[Release flow](release-flow.md) for why.

```yaml
sync-prerelease-branch:
  needs: release
  if: github.ref_name == 'main' && needs.release.outputs.release-created == 'true'
  uses: Mitchou10/github-workflow/.github/workflows/sync-prerelease-branch.yml@v0
  permissions:
    contents: write
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `RELEASE_BRANCH` | `main` | Branch to synchronise from |
| `PRERELEASE_BRANCH` | `dev` | Branch to synchronise |
| `CREATE_IF_MISSING` | `true` | Create `PRERELEASE_BRANCH` from `RELEASE_BRANCH` if it does not exist |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

## Behaviour

- Branch missing and `CREATE_IF_MISSING`: creates it from `RELEASE_BRANCH` and stops.
- Otherwise rebases `PRERELEASE_BRANCH` on `RELEASE_BRANCH` and pushes with `--force-with-lease`. In the normal
  case this is a fast-forward and nothing is rewritten.
- On conflicts, the job fails with an explicit message. Resolve them by hand: until then `dev` computes its
  versions from a stale base.

## Requirements

- Place it **last** in the pipeline, after every job that commits to `main` (a chart bump, for example).
- `dev` must accept force-pushes from `github-actions[bot]`: do not protect it against them.
- It is independent from `release-please.yml`; it can follow any release process.
