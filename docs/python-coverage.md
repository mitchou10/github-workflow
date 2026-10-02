# python-coverage.yml

Runs the unit tests with [pytest](https://docs.pytest.org/), measures the coverage with
[pytest-cov](https://pytest-cov.readthedocs.io/) and fails below a threshold. The report goes to the job summary,
and the XML and HTML reports are kept as an artifact.

```yaml
coverage:
  uses: Mitchou10/github-workflow/.github/workflows/python-coverage.yml@v0
  permissions:
    contents: read
  with:
    WORKING_DIRECTORY: backend   # optional, default: repository root
    MIN_COVERAGE: "85"           # optional, default: 80
```

The dependencies are installed with `uv sync`. pytest, pytest-cov and coverage are added on the fly: they do not
have to be dependencies of the project. The project must be managed by uv.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `WORKING_DIRECTORY` | `.` | Project directory, relative to the repository root |
| `PYTHON_VERSION` | `3.12` | Python version |
| `SYNC_ARGS` | `--all-groups` | Arguments of `uv sync`, for example `--group dev` |
| `TEST_PATH` | empty | Path of the tests. Empty lets pytest discover them |
| `PYTEST_ARGS` | empty | Extra pytest arguments, for example `-x -m "not slow"` |
| `COVERAGE_SOURCE` | empty | Comma-separated packages or directories to measure, for example `app` |
| `MIN_COVERAGE` | empty | Minimum total coverage in percent, `0` to disable |
| `BRANCH_COVERAGE` | `false` | Measure branches as well as lines |
| `COMPOSE_FILE` | empty | Docker Compose file whose services the tests need |
| `COMPOSE_SERVICES` | empty | Services to start (all of them if empty) |
| `ENV` | empty | Environment of the tests, one `KEY=value` per line. Not for secrets |
| `SETUP_COMMAND` | empty | Command run once before the tests, for example a migration |
| `UPLOAD_REPORTS` | `true` | Keep `coverage.xml` and `htmlcov` as an artifact |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

Output: `coverage`, the total in percent (for example `87`).

## Which threshold applies

In this order:

1. **`MIN_COVERAGE`**, when set (`0` turns the threshold off).
2. The project's own coverage configuration, when it has one (`[tool.coverage.*]` in `pyproject.toml`, or
   `.coveragerc`), including its `fail_under`.
3. Otherwise the default: **80 %**.

Only pytest applies the threshold. The report step never fails on it, so the summary is always written, which is
when you want to read it.

## What is measured

- With `COVERAGE_SOURCE` empty, the project code is measured, not the installed dependencies. Tests count too,
  which raises the number: prefer naming the source (`COVERAGE_SOURCE: app`) for a meaningful figure.
- `BRANCH_COVERAGE` also checks that both outcomes of each `if` were run. The number is lower than line coverage.
  Turn it on for an honest figure, and expect the first value to drop.
- Files that should not count (migrations, generated code) are excluded in the project configuration:

  ```toml
  [tool.coverage.run]
  omit = ["*/migrations/*", "tests/*"]

  [tool.coverage.report]
  fail_under = 85
  exclude_also = ["if TYPE_CHECKING:", "raise NotImplementedError"]
  ```

## Tests that need services

A reusable workflow cannot declare `services:` for its caller. Three inputs replace them:

```yaml
with:
  COMPOSE_FILE: docker-compose-test.yaml
  COMPOSE_SERVICES: postgres redis
  ENV: |
    DATABASE_URL=postgresql+asyncpg://app:app@localhost:5432/app
    REDIS_URL=redis://localhost:6379/0
  SETUP_COMMAND: uv run alembic upgrade head
```

1. The services are started with `docker compose up -d --wait`, which waits until they are healthy: give them a
   `healthcheck` in the compose file.
2. `ENV` is exported for the following steps.
3. `SETUP_COMMAND` runs once in `WORKING_DIRECTORY`, with `bash -e`.

Ports must be published (`ports: ["5432:5432"]`) so the tests reach the services on `localhost`.

## Output

- **Job summary**: total, and a table of the files below 100 % (fully covered files are hidden).
- **Log**: pytest's `term-missing` report, with the line numbers that are not covered.
- **Artifact** `coverage-<directory>-py<version>`, kept 7 days: `coverage.xml` (for Codecov, SonarQube...) and
  `htmlcov/`. Open `htmlcov/index.html` to browse the code line by line.

## Several projects in one repository

Call the workflow once per project, with its `WORKING_DIRECTORY`. Each call has its own cache, summary and artifact
(the name contains the directory). See also [path-filter](path-filter.md) to run only the projects that changed.

## Notes

- `pytest` is run directly, like `uv run pytest`. If the tests import the project code, configure the path in
  `pyproject.toml` (`[tool.pytest.ini_options]` with `pythonpath = ["."]`), or install the project as a package.
- pytest options of the project (`[tool.pytest.ini_options]`, `addopts`) are honoured.
