# lint-commits.yml

Checks that commit messages follow [Conventional Commits](https://www.conventionalcommits.org/), with
[commitlint](https://commitlint.js.org/) (version pinned in the workflow).

```yaml
name: Lint commits
on:
  pull_request:
    types: [opened, synchronize, reopened, edited]
jobs:
  lint-commits:
    uses: Mitchou10/github-workflow/.github/workflows/lint-commits.yml@v0
    permissions:
      contents: read
    with:
      LINT_PR_TITLE: true
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `ALLOWED_TYPES` | `feat,fix,docs,style,refactor,perf,test,build,ci,chore,revert` | Allowed commit types |
| `REQUIRE_SCOPE` | `false` | Require a scope, such as `feat(api)` |
| `MAX_HEADER_LENGTH` | `100` | Max length of the first line, prefix included |
| `LINT_PR_TITLE` | `false` | On pull requests, also lint the title |
| `CONFIG_FILE` | empty | Your own commitlint config, replaces the options above |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

## Which commits are checked

- Pull request: every commit between the base and the head of the PR.
- Push: the commits pushed. A new branch or a manual run checks the tip commit only.
- Merge commits (`Merge branch ...`) are ignored by commitlint.

## Squash merges

When pull requests are squash-merged, the pull request **title** becomes the commit on the target branch, and it
is what release-please reads. Set `LINT_PR_TITLE: true` and add `edited` to the `pull_request` types, so that renaming the pull request lints the title again. Otherwise a correct commit history can still produce an
invalid squashed commit.

GitHub names a pull request after its branch by default (`feat/my-branch` becomes `Feat/my branch`), which is not
a valid title: rename it before merging.

## Blocking a merge

A workflow only reports a status. To block merging, mark the job (`lint-commits / Lint commit messages`) as a
required status check in the branch protection or ruleset.

## Custom configuration

`CONFIG_FILE` takes a path in your repository, for example `commitlint.config.mjs`. It is loaded by the
pinned commitlint, so it can extend `@commitlint/config-conventional` and use its rules, but it cannot install
other plugins.
