# Release flow

Two long-lived branches:

- `main` is production. It only receives stable releases: `v0.2.0`.
- `dev` is where work lands. It produces release candidates: `v0.3.0-rc.1`, `v0.3.0-rc.2`, ...

```
feature ──PR──▶ dev ──▶ release PR (rc) merged ──▶ v0.3.0-rc.1
                 │
                 └──PR──▶ main ──▶ release PR (stable) merged ──▶ v0.3.0
                                                        │
                                       sync-prerelease-branch rebases dev on main
```

## Step by step

1. Merge work into `dev` using Conventional Commits.
2. release-please opens (or updates) a pull request `chore(dev): release X.Y.Z-rc.N` on `dev`.
   Merging it creates the tag and a GitHub pre-release.
3. When `dev` is ready for production, open a pull request `dev` → `main`.
4. On `main`, release-please opens `chore(main): release X.Y.Z`. Merging it creates the stable tag,
   the GitHub Release, and moves the floating tags `vX` and `vX.Y`.
5. The `sync-prerelease-branch` job rebases `dev` on `main`, so `dev` contains the release commit.

## Files

Two configuration/manifest pairs, one per branch:

| File | Used on | Content |
| --- | --- | --- |
| `release-please-config.json` | `main` | `release-type`, changelog sections, `initial-version` |
| `.release-please-manifest.json` | `main` | Last stable version |
| `release-please-config-rc.json` | `dev` | Same, plus `versioning: prerelease` and `prerelease-type: rc` |
| `.release-please-manifest-rc.json` | `dev` | Last rc version |

When a stable release pull request is opened on `main`, the workflow copies the stable manifest into the rc
manifest, so the next cycle on `dev` starts from the released version.

## Why the sync job matters

A release adds commits to `main` (release commit, changelog, manifest) that `dev` does not have. Without the
rebase, `dev` computes its next version from a stale base and can publish a version **lower** than the one just
released, and the next `dev` → `main` pull request conflicts on the changelog.

The job must run **after** every job that commits to `main`. It uses `GITHUB_TOKEN` on purpose: a push made
with it does not trigger workflows, so the rebase of `dev` does not start a new prerelease run.

## Without `dev`

Leave `ENABLE_PRERELEASE` off (its default). Only `main` is released, from the two files without `-rc`,
and the sync job is not needed.

## First release

release-please ignores the manifest when no release exists yet and would start at `1.0.0`. Set
`initial-version` in the configs (`0.1.0` on `main`, `0.1.0-rc.1` on `dev`), as the examples do.
