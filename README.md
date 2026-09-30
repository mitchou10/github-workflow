# github-workflow

Reusable GitHub Actions workflows, independent from any organisation's shared catalogue.

## Workflows

| Workflow | Purpose |
| --- | --- |
| [release-please.yml](.github/workflows/release-please.yml) | Release PR, tag and GitHub Release (single branch or `dev` → `main` prerelease flow) |
| [sync-prerelease-branch.yml](.github/workflows/sync-prerelease-branch.yml) | Rebases `dev` on `main` after a release |
| [python-lint.yml](.github/workflows/python-lint.yml) | `ruff check` + `ruff format --check` |
| [python-typecheck.yml](.github/workflows/python-typecheck.yml) | mypy or pyright on a uv project |

Planned: vulture.

## Usage

Call a workflow from your project with `uses: Mitchou10/github-workflow/.github/workflows/<name>.yml@v0`.
Ready-to-copy callers and configs live in [examples/](examples/).

## Versioning

This repo is versioned as a whole by release-please (see [release.yml](.github/workflows/release.yml)),
using conventional commits: `feat:` → minor, `fix:` → patch, `feat!:` / `BREAKING CHANGE` → major.
Every release moves the floating tags `vX` and `vX.Y`, so callers can pin `@v0` (auto-updates),
`@v0.1` or an exact `@v0.1.2`. A change to any workflow's inputs/outputs that breaks callers must be
committed as a breaking change.

### python-lint / python-typecheck

Both run at the repository root by default; pass `WORKING_DIRECTORY` to target a sub-project.
See [examples/python/caller.yml](examples/python/caller.yml) for all inputs. Ruff reads the config of
`pyproject.toml` / `ruff.toml` in that directory. The typecheck needs a uv project (`uv sync`).

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
