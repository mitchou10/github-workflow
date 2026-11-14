# release-helm.yml

Packages a chart and pushes it to an **OCI registry** (ghcr.io by default), with the version its `Chart.yaml` says.
Made for an application repository, where the GitHub Releases and the tags belong to the application: the chart is
published to the registry only, and installed with `helm install oci://...`.

```yaml
helm:
  needs: release                       # a job that tells a release was made
  if: needs.release.outputs.release-created == 'true'
  uses: Mitchou10/github-workflow/.github/workflows/release-helm.yml@v0
  permissions:
    contents: read
    packages: write
  with:
    CHART_PATH: charts/app
    REPOSITORY: my-org/charts          # ghcr.io/my-org/charts/app
```

The chart version is the one in `Chart.yaml`. Let release-please bump it with the application (`extra-files` of type
`yaml` with the paths `$.version` and `$.appVersion` on the `Chart.yaml`), so that the chart of a
release deploys the images of that release.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `CHART_PATH` | empty | One chart, by its directory (the one holding `Chart.yaml`) |
| `CHARTS_DIR` | `./charts` | Without `CHART_PATH`: the directory whose subdirectories are charts |
| `CHART_NAME` | empty | Without `CHART_PATH`: only this subdirectory of `CHARTS_DIR` |
| `CHART_VERSION` | empty | Stamp this version instead of the one of `Chart.yaml` |
| `APP_VERSION` | empty | Stamp this `appVersion` instead of the one of `Chart.yaml` |
| `CHECKOUT_REF` | empty | Ref to check out (default: the commit that triggered the run) |
| `HELM_REPOS` | empty | Repositories of the dependencies, `name=url`, comma separated |
| `REGISTRY` | `ghcr.io` | Registry. Another one needs `REGISTRY_USERNAME` and `REGISTRY_PASSWORD` |
| `REPOSITORY` | the repository | Path in the registry. Lower-cased |
| `ALLOW_OVERWRITE` | `false` | Push a version that is already published |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

`CHART_PATH` and `CHART_NAME` are exclusive.

## Output

`published-charts`: a JSON array of what was pushed, `[{name, version, repository, digest}]`. Give it to
[attest-helm](attest-helm.md) to sign the charts.

## Good to know

- **A version is never pushed twice** unless `ALLOW_OVERWRITE` is set: a consumer pins a chart by version, and a
  signature is bound to the digest first pushed. A second push of a version fails the run, before anything is pushed.
- The push is retried (three attempts), because ghcr.io sometimes answers the tag step of a first push with a 404.
- A prerelease (`0.3.0-rc.1`) is an ordinary version for Helm; `helm install` skips it unless `--devel` is given.

## Using the chart

```sh
helm install app oci://ghcr.io/my-org/charts/app --version 1.2.3
```
