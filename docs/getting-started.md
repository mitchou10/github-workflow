# Getting started

## Calling a workflow

The workflows are reusable (`workflow_call`). From your project:

```yaml
jobs:
  lint:
    uses: Mitchou10/github-workflow/.github/workflows/python-lint.yml@v0
    permissions:
      contents: read
```

Ready-to-copy callers are in [examples/](../examples/).

The repository must be reachable from the calling one: public, or private with
Settings → Actions → General → Access set to allow other repositories.

## Pinning a version

This repository is versioned as a whole (see [Release flow](release-flow.md)).

| Reference | Behaviour |
| --- | --- |
| `@v0` | Follows every `0.x` release. Recommended. |
| `@v0.2` | Follows patch releases of `0.2`. |
| `@v0.2.0` | Never changes. |
| `@dev` | Release candidates, may break. To try unreleased changes. |
| `@main` | Same content as the latest release. |

Once the repository reaches `1.0.0`, use `@v1`.

## Repository settings

Needed once per project that uses `release-please`:

1. Settings → Actions → General → Workflow permissions → **Read and write permissions**.
2. Same page → **Allow GitHub Actions to create and approve pull requests**.

Without the second one, release-please fails when opening the release pull request
(see [Troubleshooting](troubleshooting.md)).

## Commit messages

Versions and changelogs are computed from [Conventional Commits](https://www.conventionalcommits.org/):

| Prefix | Effect |
| --- | --- |
| `fix:` | patch |
| `feat:` | minor |
| `feat!:` or a `BREAKING CHANGE:` footer | major (minor while below `1.0.0` only if configured) |
| `docs:`, `chore:`, `ci:`, `test:`, `style:` | no release by themselves |

Use the [lint-commits](lint-commits.md) workflow to enforce it.
