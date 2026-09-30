# Documentation

| Page | Content |
| --- | --- |
| [Getting started](getting-started.md) | Using the workflows in a project, pinning versions, repository settings |
| [Release flow](release-flow.md) | `main` / `dev`, release candidates, how versions are computed |
| [release-please](release-please.md) | Inputs, outputs, authentication, configuration files |
| [sync-prerelease-branch](sync-prerelease-branch.md) | Keeping `dev` on top of `main` after a release |
| [python-lint](python-lint.md) | ruff, default rules and overrides |
| [python-typecheck](python-typecheck.md) | ty, mypy or pyright |
| [python-deadcode](python-deadcode.md) | vulture, confidence levels, false positives |
| [scan-gitleaks](scan-gitleaks.md) | gitleaks, what is scanned, allowlists |
| [scan-trivy](scan-trivy.md) | Trivy: fs, config and image scans |
| [lint-helm](lint-helm.md) | Helm chart lint, helm-docs check |
| [build-docker](build-docker.md) | Build and push images, tags, multi-platform |
| [path-filter](path-filter.md) | Run jobs only when their folder changed |
| [status-gate](status-gate.md) | One required check for the whole workflow |
| [Dependency updates](dependency-updates.md) | Dependabot or Renovate, and the shared preset |
| [lint-commits](lint-commits.md) | Conventional Commits check |
| [Troubleshooting](troubleshooting.md) | Errors met while setting things up |

Every input is optional: a workflow called without `with:` uses its defaults.
