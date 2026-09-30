# python-typecheck.yml

Runs a type checker over a [uv](https://docs.astral.sh/uv/) project. The dependencies are installed first
(`uv sync`), so imports resolve; the checker itself is added on the fly and does not have to be a dependency.

```yaml
typecheck:
  uses: Mitchou10/github-workflow/.github/workflows/python-typecheck.yml@v0
  permissions:
    contents: read
  with:
    WORKING_DIRECTORY: backend   # optional, default: repository root
    TARGET: app                  # optional, default: .
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `WORKING_DIRECTORY` | `.` | Project directory, relative to the repository root |
| `PYTHON_VERSION` | `3.12` | Python used for the environment and the checker |
| `TYPE_CHECKER` | `ty` | `ty`, `mypy` or `pyright` |
| `TARGET` | `.` | Path given to the checker, relative to `WORKING_DIRECTORY` |
| `SYNC_ARGS` | `--all-groups` | Arguments of `uv sync`, for example `--group dev` or `--extra api` |
| `EXTRA_ARGS` | empty | Extra arguments for the checker |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

## Checkers

| Checker | Configuration read from | Note |
| --- | --- | --- |
| `ty` | `[tool.ty]` in `pyproject.toml` or `ty.toml` | Astral's checker, still `0.x`: expect changes |
| `mypy` | `[tool.mypy]`, `mypy.ini` | Mature; pass `--strict` in `EXTRA_ARGS` if wanted |
| `pyright` | `[tool.pyright]`, `pyrightconfig.json` | |

The version of the checker is not pinned: it is the latest release.

## Requirements

The project must be managed by uv (`pyproject.toml`, ideally `uv.lock`). The uv cache is keyed on
`WORKING_DIRECTORY/uv.lock`.
