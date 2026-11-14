# lint-helm-render.yml

Renders a chart with `helm template` and checks each manifest against the Kubernetes API schemas with
[kubeconform](https://github.com/yannh/kubeconform). `helm lint` says the templates render; this says that what they
render is a valid Kubernetes object (a field misspelled, a wrong type, a removed API version).

```yaml
lint-helm-render:
  uses: Mitchou10/github-workflow/.github/workflows/lint-helm-render.yml@v0
  permissions:
    contents: read
  with:
    CHART_PATH: charts/app
    VALUES_FILES: ci/default-values.yaml,ci/ingress-values.yaml
```

One render per values file, plus one with the default values alone. A chart that refuses its default values (it needs a
secret, say) is rendered with `DEFAULT_VALUES: false` and values files that give it what it needs.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `CHART_PATH` | required | Directory of the chart (the one holding `Chart.yaml`) |
| `VALUES_FILES` | empty | Values files, comma separated, relative to `CHART_PATH`. Each is a render of its own |
| `DEFAULT_VALUES` | `true` | Also render with the default values alone |
| `KUBERNETES_VERSION` | `1.31.0` | Version of the API schemas |
| `KUBECONFORM_VERSION` | `v0.7.0` | kubeconform version |
| `HELM_VERSION` | `latest` | Helm version |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

Resources of a kind kubeconform has no schema for (a custom resource, the Gateway API) are skipped
(`-ignore-missing-schemas`), not failed.
