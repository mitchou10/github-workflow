# Troubleshooting

## `GitHub Actions is not permitted to create or approve pull requests`

release-please cannot open the release pull request. Settings → Actions → General → Workflow permissions →
enable **Allow GitHub Actions to create and approve pull requests**. Then re-run the failed run.

## The first release is `1.0.0` instead of `0.1.0` (or has no `-rc`)

With no release yet, release-please ignores the manifest and starts at `1.0.0`. Set `initial-version` in
the config (`0.1.0`, and `0.1.0-rc.1` in the `-rc` config). If a wrong pull request is already open, close it,
delete its `release-please--branches--*` branch, fix the config and push again.

## A release pull request title still shows the old version

release-please updates the content of the branch but does not always rewrite the title. The tag created on
merge uses the content. To get a clean title, close the pull request and delete its branch; the next push
recreates it.

## The release pull request has no checks

It was opened with `GITHUB_TOKEN`, which cannot trigger workflows. Pass `APP_CLIENT_ID` and `APP_PRIVATE_KEY`
(or `GH_PAT`) to the release workflow. See [release-please](release-please.md#which-token-and-why-it-matters).

## No release pull request appears

- The commits since the last release are only `docs:`, `chore:`, `ci:`... which do not trigger a release.
- Commits are not Conventional Commits: check them with [lint-commits](lint-commits.md).
- The run is on a branch that is neither `RELEASE_BRANCH` nor `PRERELEASE_BRANCH`.

## `Could not find releases` / `looking for tagName: v0.0.0` in the logs

Expected before the first release. It is not an error.

## The sync job fails with a rebase conflict

`dev` and `main` diverged (typically a change made directly on `main`). Rebase `dev` on `main` locally, resolve
the conflicts and force-push `dev` with `--force-with-lease`.

## `uses:` cannot find the workflow

- The repository is private and does not allow access from other repositories.
- The reference does not exist yet: `@v0` only exists after the first stable release. Use `@main` or `@dev`.
- The path or the file name is wrong: the workflows live in `.github/workflows/`.

## `ruff` reports rules the project did not ask for

The project has no ruff config in `WORKING_DIRECTORY`, so the defaults (`E,F,I,UP,B`, line length 120) apply.
Add a `[tool.ruff]` table, or set `RULES`, `IGNORE` and `LINE_LENGTH`.

## Workflow fails to start: `requesting 'security-events: write', but is only allowed 'security-events: none'`

A workflow called with `SECURITY_TAB` needs `security-events: write` on the **calling job**:

```yaml
permissions:
  contents: read
  security-events: write
```

## `Resource not accessible by integration` when uploading to the Security tab

Pull requests from forks get a read-only token and cannot upload. Private repositories also need GitHub
Advanced Security for code scanning. The scan itself still runs and `FAIL_ON_LEAKS` / `FAIL_ON_FINDINGS` still apply.

## `No chart changes detected` / no chart linted

chart-testing only looks at the **subdirectories** of `CHART_DIRS`, and at changed charts when `ONLY_CHANGED` is
set. Check that `Chart.yaml` is in a subdirectory (`helm/Chart.yaml`, not `./Chart.yaml`) and that `CHART_DIRS`
points at its parent. See [lint-helm](lint-helm.md#where-charts-are-found).

## `targetBranch 'origin/main' does not exist`

`ONLY_CHANGED` compares with a remote branch that is not there. Set `TARGET_BRANCH` to an existing branch of the
repository.
