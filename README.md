# github-workflow

Reusable GitHub Actions workflows, independent from any organisation's shared catalogue.

## Workflows

| Workflow | Purpose |
| --- | --- |
| [release-please.yml](.github/workflows/release-please.yml) | Release PR, tag and GitHub Release (single branch or `dev` → `main` prerelease flow) |
| [sync-prerelease-branch.yml](.github/workflows/sync-prerelease-branch.yml) | Rebases `dev` on `main` after a release |
| [python-lint.yml](.github/workflows/python-lint.yml) | `ruff check` + `ruff format --check` |
| [python-typecheck.yml](.github/workflows/python-typecheck.yml) | ty (default), mypy or pyright on a uv project |
| [scan-gitleaks.yml](.github/workflows/scan-gitleaks.yml) | gitleaks: leaked secrets in the git history (optional Security tab upload) |
| [scan-trivy.yml](.github/workflows/scan-trivy.yml) | Trivy: dependencies, misconfigurations, container images |
| [lint-helm.yml](.github/workflows/lint-helm.yml) | chart-testing + helm-docs: Helm chart lint |
| [lint-commits.yml](.github/workflows/lint-commits.yml) | commitlint: commits follow Conventional Commits |
| [python-deadcode.yml](.github/workflows/python-deadcode.yml) | vulture: unused code |

## Documentation

Full documentation is in [docs/](docs/README.md): [getting started](docs/getting-started.md),
[release flow](docs/release-flow.md), one page per workflow, and [troubleshooting](docs/troubleshooting.md).

## Usage

Call a workflow from your project with `uses: Mitchou10/github-workflow/.github/workflows/<name>.yml@v0`.
Ready-to-copy callers and configs live in [examples/](examples/).

## Versioning

This repo is versioned as a whole by release-please (see [release.yml](.github/workflows/release.yml)),
using conventional commits: `feat:` → minor, `fix:` → patch, `feat!:` / `BREAKING CHANGE` → major.
Every release moves the floating tags `vX` and `vX.Y`, so callers can pin `@v0` (auto-updates),
`@v0.1` or an exact `@v0.1.2`. A change to any workflow's inputs/outputs that breaks callers must be
committed as a breaking change.

### lint-commits

Runs commitlint (pinned) on the commits of a pull request, or of a push. Add it on `pull_request`, see
[examples/commits/caller.yml](examples/commits/caller.yml). Set `LINT_PR_TITLE` with squash merges. Allowed types,
scope requirement and header length are inputs; `CONFIG_FILE` swaps in your own commitlint config.
To block merging on failure, mark the job as a required status check in the branch protection.

### python-lint / python-typecheck / python-deadcode

Both run at the repository root by default; pass `WORKING_DIRECTORY` to target a sub-project.
See [examples/python/caller.yml](examples/python/caller.yml) for all inputs.

Ruff rules: if the project has its own config (`ruff.toml`, `.ruff.toml` or `[tool.ruff]` in `pyproject.toml`),
it is used as is. Otherwise the defaults apply (`RULES` = `E,F,I,UP,B`, `LINE_LENGTH` = `120`). Setting the
`RULES`, `IGNORE`, `LINE_LENGTH` or `CONFIG_FILE` inputs overrides either.
 The typecheck needs a uv project (`uv sync`).

### release-please

1. Copy [caller.yml](examples/release-please/caller.yml) to `.github/workflows/release.yml`.
2. Copy [release-please-config.json](examples/release-please/release-please-config.json) and
   [.release-please-manifest.json](examples/release-please/.release-please-manifest.json) to the repo root.
3. Use conventional commits (`feat:`, `fix:`, …).
4. Repo settings → Actions → allow "Read and write permissions" and "Allow GitHub Actions to create and approve pull requests".

Branches: `main` is production (stable `vX.Y.Z`), `dev` carries release candidates (`vX.Y.Z-rc.N`).
Copy also the `-rc` config/manifest, and keep the `sync-prerelease-branch` job last in the caller: after a
release it rebases `dev` on `main`. Create `dev` from `main` before the first run (the sync job also does it).

Auth: by default `GITHUB_TOKEN` is used, so the release PR does not trigger CI. Pass `APP_CLIENT_ID` +
`APP_PRIVATE_KEY` (or `GH_PAT`) to make it trigger checks.

Outputs: `release-created`, `version`, `tag-name`, `pr-created`.
