# release-please.yml

Opens and updates the release pull request; once it is merged, creates the tag and the GitHub Release.

```yaml
jobs:
  release:
    uses: Mitchou10/github-workflow/.github/workflows/release-please.yml@v0
    permissions:
      contents: write
      issues: write
      pull-requests: write
    with:
      ENABLE_PRERELEASE: true
```

Caller with the sync job: [examples/release-please/caller.yml](../examples/release-please/caller.yml).

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `RELEASE_BRANCH` | `main` | Branch of stable releases |
| `PRERELEASE_BRANCH` | `dev` | Branch of release candidates (needs `ENABLE_PRERELEASE`) |
| `ENABLE_PRERELEASE` | `false` | Use the `-rc` config and manifest on `PRERELEASE_BRANCH` |
| `RELEASE_CONFIG_FILE` | `release-please-config.json` | Config for the release branch |
| `RELEASE_MANIFEST_FILE` | `.release-please-manifest.json` | Manifest for the release branch |
| `PRERELEASE_CONFIG_FILE` | `release-please-config-rc.json` | Config for the prerelease branch |
| `PRERELEASE_MANIFEST_FILE` | `.release-please-manifest-rc.json` | Manifest for the prerelease branch |
| `TAG_MAJOR_AND_MINOR` | `false` | Move `vX` and `vX.Y` on a stable release (never on a release candidate) |
| `AUTOMERGE` | `false` | Queue the release pull request for auto-merge (needs an App or PAT, and "Allow auto-merge") |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

## Outputs

| Output | Description |
| --- | --- |
| `release-created` | `true` when a release was created |
| `version` | Full version, for example `0.3.0-rc.1` |
| `tag-name` | Tag, for example `v0.3.0-rc.1` |
| `pr-created` | `true` when a release pull request was created or updated |

Chain your own jobs on them:

```yaml
build:
  needs: release
  if: needs.release.outputs.release-created == 'true'
```

## Secrets

All optional.

| Secret | Description |
| --- | --- |
| `APP_CLIENT_ID` + `APP_PRIVATE_KEY` | GitHub App credentials. Preferred. Must be given together. |
| `GH_PAT` | Personal access token, alternative to the App. |

Pass them with `secrets:` (or `secrets: inherit`).

### Which token, and why it matters

With none of them, `GITHUB_TOKEN` is used and everything works, **except** that the release pull request does
not trigger `pull_request` workflows: its checks never run. If you require status checks on `main` / `dev`, the
release pull request cannot be merged. Use an App or a PAT then.

The `git push` steps of the workflow (floating tags, manifest sync) deliberately keep using `GITHUB_TOKEN` when
possible so they cannot re-trigger your pipeline.

## Configuration files

Start from [examples/release-please/](../examples/release-please/):

- `release-please-config.json`, `.release-please-manifest.json` for `main`;
- the `-rc` twins for `dev`.

`release-type: simple` keeps the version in a manifest and the changelog only. To also bump a version inside a
file, add `extra-files` (see the
[release-please documentation](https://github.com/googleapis/release-please/blob/main/docs/manifest-releaser.md)),
for example:

```json
"extra-files": [
  { "type": "toml", "path": "pyproject.toml", "jsonpath": "project.version" }
]
```

`changelog-sections` decides which commit types appear in the changelog and under which heading.
