# scan-trivy.yml

Scans with [Trivy](https://trivy.dev/): the repository files, its infrastructure configuration, or a container
image.

```yaml
trivy-fs:
  uses: Mitchou10/github-workflow/.github/workflows/scan-trivy.yml@v0
  permissions:
    contents: read

trivy-config:
  uses: Mitchou10/github-workflow/.github/workflows/scan-trivy.yml@v0
  permissions:
    contents: read
  with:
    SCAN_TYPE: config
    TARGET: helm/

trivy-image:
  uses: Mitchou10/github-workflow/.github/workflows/scan-trivy.yml@v0
  permissions:
    contents: read
    packages: read           # private image on ghcr.io
  with:
    SCAN_TYPE: image
    TARGET: ghcr.io/owner/app:1.2.3
```

## Scan types

| `SCAN_TYPE` | Looks for | `TARGET` |
| --- | --- | --- |
| `fs` (default) | Vulnerable dependencies (lockfiles) and secrets | A path, default `.` |
| `config` | Misconfigurations in Dockerfile, Kubernetes, Helm, Terraform... | A path, default `.` |
| `image` | Vulnerabilities in the OS packages and libraries of an image | An image reference (required) |

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `SCAN_TYPE` | `fs` | `fs`, `config` or `image` |
| `TARGET` | `.` | Path, or image reference for `image` |
| `SEVERITY` | `CRITICAL,HIGH` | Severities reported (and that fail the job) |
| `IGNORE_UNFIXED` | `true` | Skip vulnerabilities that have no fix yet |
| `SCANNERS` | empty | `vuln`, `secret`, `misconfig`, `license`, comma separated. Empty: Trivy's default for the type |
| `TRIVYIGNORES` | empty | Ignore file(s). Empty uses `.trivyignore.yaml` if it exists |
| `CONFIG_FILE` | empty | A `trivy.yaml` configuration file |
| `TRIVY_VERSION` | `v0.70.0` | Trivy version |
| `TIMEOUT` | empty | For example `15m`. Empty uses Trivy's 5 minutes |
| `SECURITY_TAB` | `false` | Upload the findings to the GitHub Security tab |
| `CATEGORY` | empty | Code scanning category. Empty means `trivy-<SCAN_TYPE>` |
| `FAIL_ON_FINDINGS` | `true` | Fail when a finding matches `SEVERITY` |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

Secrets, both optional: `REGISTRY_USERNAME` and `REGISTRY_PASSWORD`, to pull a private image outside `ghcr.io`.
On `ghcr.io` the workflow signs in with the job token.

Output: `findings` (`true` when something matched `SEVERITY`).

## Permissions

The workflow declares none: it gets those of the calling job.

| Use | The calling job needs |
| --- | --- |
| Default | `contents: read` |
| `SECURITY_TAB: true` | plus `security-events: write` |
| Private image on `ghcr.io` | plus `packages: read` |

## Output

- Default: a table in the job summary.
- `SECURITY_TAB: true`: SARIF uploaded to Security → Code scanning; the summary shows the number of findings.
  The severity filter is applied to the SARIF as well. Private repositories need GitHub Advanced Security. If the
  upload fails the job warns, and `FAIL_ON_FINDINGS` still applies.
- Scanning several images with `SECURITY_TAB`: give each a distinct `CATEGORY` (the image name, for example),
  otherwise the uploads replace each other and only the last one remains.

## Ignoring findings

Put accepted findings in `.trivyignore.yaml`, with a reason for each:

```yaml
vulnerabilities:
  - id: CVE-2024-12345
    paths:
      - "app/legacy/"
    statement: Not reachable, the vulnerable function is never called.
misconfigurations:
  - id: KSV-0110
    statement: Portable chart, the namespace is chosen at install time.
```

The YAML form supports scoping by path and a documented reason. A plain `.trivyignore` (one ID per line) is read
by Trivy on its own. Prefer ignoring by ID and path over lowering `SEVERITY`.

## Notes

- The vulnerability database is downloaded on every run. The job token is used to lower the chance of hitting the
  GitHub API rate limit; if it happens anyway, retry.
- A scan that cannot run (unknown image, timeout) fails the job even with `FAIL_ON_FINDINGS: false`: only findings
  are tolerated, never a broken scan. Raise `TIMEOUT` for large images.
- Scanning an image built in the same pipeline: push it first (the workflow pulls it from a registry).
