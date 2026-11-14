# attest-helm.yml

Signs the charts [release-helm](release-helm.md) pushed, and/or attaches a build provenance to them. A signature
says the chart was made by this repository's workflow; the provenance says from which commit.

```yaml
attest-helm:
  needs: helm                          # the job that called release-helm.yml
  uses: Mitchou10/github-workflow/.github/workflows/attest-helm.yml@v0
  permissions:
    packages: write
    id-token: write
    attestations: write
  with:
    CHARTS: ${{ needs.helm.outputs.published-charts }}
    SIGN: true
    PROVENANCE: true
```

It is a separate workflow because both need `id-token: write`, and GitHub checks a reusable workflow's whole job graph
against what the caller grants: folding it into release-helm would ask every caller for it.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `CHARTS` | required | The `published-charts` output of release-helm. An empty array does nothing |
| `SIGN` | `false` | Sign the digest with [cosign](https://github.com/sigstore/cosign), keyless |
| `PROVENANCE` | `false` | Attach a SLSA build provenance (GitHub's `attest-build-provenance`) |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

At least one of `SIGN` and `PROVENANCE` must be on, or the run fails (it would look like it attested everything).
Each chart is attested **by digest**, never by tag. All the charts must be in the same registry.

## Verifying

```sh
cosign verify ghcr.io/my-org/charts/app:1.2.3 \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  --certificate-identity-regexp '^https://github\.com/Mitchou10/github-workflow/\.github/workflows/attest-helm\.yml@.+$'
```

The identity of a keyless signature is the workflow that made it: here the shared `attest-helm.yml`, not the caller.
