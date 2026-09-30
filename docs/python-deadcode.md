# python-deadcode.yml

Finds unused code (imports, functions, classes, variables, unreachable code) with
[vulture](https://github.com/jendrikseipp/vulture). Vulture is static: it is run with `uvx` and needs no
dependencies. The job fails when something is reported (exit code 3).

```yaml
deadcode:
  uses: Mitchou10/github-workflow/.github/workflows/python-deadcode.yml@v0
  permissions:
    contents: read
  with:
    WORKING_DIRECTORY: backend   # optional, default: repository root
    PATHS: app                   # optional, default: .
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `WORKING_DIRECTORY` | `.` | Directory to analyse, relative to the repository root |
| `PATHS` | `.` | Space-separated paths, relative to `WORKING_DIRECTORY`. Overrides `paths` of the project config |
| `PYTHON_VERSION` | `3.12` | Python used to run vulture |
| `VULTURE_VERSION` | latest | Pin vulture, for example `2.14` |
| `MIN_CONFIDENCE` | empty | Minimum confidence, 0 to 100 |
| `EXCLUDE` | empty | Comma-separated path patterns to skip |
| `IGNORE_NAMES` | empty | Names to ignore, wildcards allowed (`visit_*,Meta`) |
| `IGNORE_DECORATORS` | empty | Decorated functions considered used (`@app.route,@pytest.fixture`) |
| `WHITELIST` | empty | Whitelist file(s), space separated |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

## Which options apply

1. The project has a `[tool.vulture]` table in `pyproject.toml` (in `WORKING_DIRECTORY`): it is used as is.
2. Otherwise the **defaults** apply: `MIN_CONFIDENCE` = `80`, and `EXCLUDE` = virtualenv, `node_modules` and
   `site-packages` directories.
3. `MIN_CONFIDENCE`, `EXCLUDE`, `IGNORE_NAMES`, `IGNORE_DECORATORS`, `WHITELIST` and `PATHS`, when set, are applied
   on top and win over both.

## Confidence levels

Vulture cannot be certain that something is unused, so it gives each finding a confidence:

| Confidence | What |
| --- | --- |
| 100 | Unreachable code, unused arguments |
| 90 | Unused imports |
| 60 | Unused functions, classes, methods, attributes, variables |

At the default of `80` only certain findings are reported: **unused imports and unreachable code**. Lower
`MIN_CONFIDENCE` to `60` to also find unused functions, classes and variables. Expect false positives there,
because vulture cannot see dynamic use: framework callbacks (route handlers, Pydantic validators, Django
models), `getattr`, plugin entry points.

## Handling false positives

In order of preference:

- `IGNORE_DECORATORS`: for everything registered by a decorator, for example `@app.route,@router.get`.
- `IGNORE_NAMES`: for names a framework calls, for example `Meta,model_config,visit_*`.
- `EXCLUDE`: for generated code, for example `*/migrations/*`.
- `WHITELIST`: a Python file listing names that count as used. Generate a first one with
  `uvx vulture . --make-whitelist > whitelist.py` and prune it.

A project that wants stable rules should put them in `pyproject.toml`, where they also apply locally:

```toml
[tool.vulture]
min_confidence = 60
paths = ["app"]
exclude = ["*/migrations/*"]
ignore_decorators = ["@router.get", "@router.post"]
ignore_names = ["Meta", "model_config"]
```

Vulture then reports the same results locally (`uvx vulture`) and in CI.
