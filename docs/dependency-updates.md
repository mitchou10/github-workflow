# Keeping dependencies up to date

The workflows pin actions to a commit SHA (`uses: actions/checkout@3d3c42e...  # v7.0.1`). It protects against a
tag being moved, but the pins have to be updated. Two tools do it. **Use one of them per repository, not both:**
they would open duplicate pull requests for the same update.

| | Dependabot | Renovate |
| --- | --- | --- |
| Setup | A file, nothing to install | Install the [Renovate GitHub App](https://github.com/apps/renovate) |
| Updates SHA-pinned actions and their `# vX.Y.Z` comment | Yes | Yes (`helpers:pinGitHubActionDigests`) |
| Tool versions in workflow inputs (gitleaks, trivy...) | No | Yes, with annotations |
| Shared configuration between repositories | No, one file per repository | Yes, a preset |
| Other ecosystems (pip, npm, docker, helm) | Yes | Yes, more of them |

Pick Dependabot for its zero setup. Pick Renovate to share one configuration across projects and to keep the
tool versions current.

## Dependabot

Copy [examples/dependabot/dependabot.yml](../examples/dependabot/dependabot.yml) to `.github/dependabot.yml`. This
repository uses it. Two choices in the file:

- `commit-message.prefix: ci` makes the titles `ci: bump ...`. Without it they are `Bump x from a to b`, which
  [lint-commits](lint-commits.md) rejects. `ci` is not a releasable type, so an update alone does not cut a release.
- `target-branch: dev` sends the pull requests to the prerelease branch; remove it if you release from `main` only.

Add the ecosystems you use:

```yaml
  - package-ecosystem: uv        # or pip
    directory: /
    commit-message: { prefix: build }
  - package-ecosystem: docker
    directory: /
    commit-message: { prefix: build }
```

## Renovate

Install the app on the repository, then add a `renovate.json`
([example](../examples/renovate/renovate.json)):

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>Mitchou10/github-workflow//renovate/default"],
  "baseBranches": ["dev"]
}
```

The shared preset [renovate/default.json](../renovate/default.json) pins GitHub Actions to digests, groups their
updates in one pull request, uses Conventional Commits (`ci(deps): ...`), and tracks annotated tool versions.

### Annotated versions

A workflow input whose default is a tool version can be tracked with a comment above it:

```yaml
      TRIVY_VERSION:
        # renovate: datasource=github-releases depName=aquasecurity/trivy
        default: v0.70.0
```

The workflows of this repository are annotated this way (`GITLEAKS_VERSION`, `TRIVY_VERSION`,
`HELM_DOCS_VERSION`). In your own projects, annotate the versions you pass to `with:` in the same way.

## Being merged

Both tools open pull requests that go through your CI, including [lint-commits](lint-commits.md) and the
[status gate](status-gate.md). With `GITHUB_TOKEN` the release pull request does not trigger checks, but
Dependabot and Renovate pull requests do.
