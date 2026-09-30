# python-lint.yml

Runs `ruff check` and `ruff format --check`. Ruff is run with `uvx`: nothing to install in the project.

```yaml
lint:
  uses: Mitchou10/github-workflow/.github/workflows/python-lint.yml@v0
  permissions:
    contents: read
  with:
    WORKING_DIRECTORY: backend   # optional, default: repository root
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `WORKING_DIRECTORY` | `.` | Directory to lint, relative to the repository root |
| `PYTHON_VERSION` | `3.12` | Python used to run ruff |
| `RUFF_VERSION` | latest | Pin ruff, for example `0.9.10` |
| `FORMAT_CHECK` | `true` | Also run `ruff format --check` |
| `RULES` | empty | Rule selectors (`--select`), comma separated |
| `IGNORE` | empty | Rules to ignore (`--ignore`), comma separated |
| `LINE_LENGTH` | empty | Line length |
| `CONFIG_FILE` | empty | Ruff config file to force, relative to `WORKING_DIRECTORY` |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

## Which rules apply

In this order:

1. **`CONFIG_FILE`** is set: that file is used.
2. The project has its own config (`ruff.toml`, `.ruff.toml`, or a `[tool.ruff]` table in `pyproject.toml`
   in `WORKING_DIRECTORY`): it is used as is.
3. Otherwise the **defaults** apply: rules `E,F,I,UP,B` and line length `120`.

Then `RULES`, `IGNORE` and `LINE_LENGTH`, when set, are applied on top of whichever of the three won.

| Default rule set | Meaning |
| --- | --- |
| `E` | pycodestyle errors |
| `F` | pyflakes (unused imports, undefined names) |
| `I` | import sorting |
| `UP` | pyupgrade |
| `B` | flake8-bugbear |

Examples:

```yaml
with:
  RULES: "E,F,I,UP,B,SIM"   # more rules than the default
  IGNORE: "E501"            # turn one off
```

`--select` and `--ignore` only apply to `ruff check`; `ruff format` does not accept them.
The formatter has no rules, only its config and the line length.

The project config is only detected in `WORKING_DIRECTORY`. A config in a parent directory is still found by
ruff itself, but the defaults will then also be applied: put a config next to the code, or use `CONFIG_FILE`.
