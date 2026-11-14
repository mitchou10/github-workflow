# lint-helm-schema.yml

Checks values files of a chart against the chart's own `values.schema.json`, with
[check-jsonschema](https://github.com/python-jsonschema/check-jsonschema). `helm lint` already applies the schema
to the chart's values; this checks the **other** files (`ci/*-values.yaml`, per-environment files) without installing anything.

```yaml
lint-helm-schema:
  uses: Mitchou10/github-workflow/.github/workflows/lint-helm-schema.yml@v0
  permissions:
    contents: read
  with:
    CHART_PATH: charts/app
    VALUES_FILES: values.yaml,ci/default-values.yaml
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `CHART_PATH` | required | Directory of the chart (the one holding the schema) |
| `SCHEMA_FILE` | `values.schema.json` | Schema file, relative to `CHART_PATH` |
| `VALUES_FILES` | `values.yaml` | Values files to check, comma separated, relative to `CHART_PATH`. A file that does not exist is skipped with a warning |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

The job fails if the schema is missing, or if none of the values files exists.
