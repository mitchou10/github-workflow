# scan-gitleaks.yml

Looks for leaked secrets (API keys, tokens, private keys...) in the git history with
[gitleaks](https://github.com/gitleaks/gitleaks).

```yaml
gitleaks:
  uses: Mitchou10/github-workflow/.github/workflows/scan-gitleaks.yml@v0
  permissions:
    contents: read
    security-events: write   # only with SECURITY_TAB: true
  with:
    SECURITY_TAB: true       # optional
```

## Permissions

The workflow declares no permissions of its own: it gets those of the calling job. Declaring
`security-events: write` in the workflow would force every caller to grant it, even those that never upload.

| Use | The calling job needs |
| --- | --- |
| Default | `contents: read` |
| `SECURITY_TAB: true` | `contents: read` and `security-events: write` |

The workflow installs the gitleaks CLI (MIT), pinned and checked against the release checksums. It does not use
`gitleaks-action`, which needs a paid licence on organisation repositories.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `GITLEAKS_VERSION` | `8.30.1` | Gitleaks version |
| `CONFIG_FILE` | empty | Config file, relative to the repository root |
| `FULL_HISTORY` | `false` | Scan the whole history of the checked-out ref |
| `LOG_OPTS` | empty | Revision range for `git log`, for example `--all`. Replaces the automatic range |
| `SECURITY_TAB` | `false` | Upload the findings to the GitHub Security tab |
| `CATEGORY` | `gitleaks` | Code scanning category of the upload |
| `FAIL_ON_LEAKS` | `true` | Fail the workflow when a leak is found |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

Output: `leaks` (`true` / `false`).

## What is scanned

| Event | Scanned |
| --- | --- |
| Pull request | The commits of the PR (`base..head`) |
| Push | The pushed commits (`before..after`) |
| New branch, force-push, manual run | The whole history of `HEAD` (the base is unknown) |
| `FULL_HISTORY: true` | The whole history of `HEAD` |
| `LOG_OPTS` set | Whatever it says, for example `--all` for every branch |

Scanning only the new commits keeps the job fast and stops an old, already accepted finding from failing every
new pull request. Schedule a `FULL_HISTORY` run (a `schedule:` trigger) to re-check the whole history now and
then.

## Rules

- Without configuration: gitleaks' default rules.
- A `.gitleaks.toml` at the repository root is picked up automatically.
- `CONFIG_FILE` points to another file.

To keep the default rules and add your own or allow some findings, start the file with:

```toml
[extend]
useDefault = true
```

## Handling a finding

1. **Rotate the secret.** Deleting it from the code is not enough: it stays in the history and must be considered
   compromised.
2. If it is a false positive (a test key, a fixture), allow it in `.gitleaks.toml` by rule, path and line:

   ```toml
   [[allowlists]]
   targetRules = ["generic-api-key"]
   description = "Test key, not used anywhere."
   condition = "AND"
   paths = ['''^tests/fixtures/''']
   ```

   Prefer this to fingerprints in `.gitleaksignore`: a fingerprint contains the commit SHA, so it stops
   matching after a rebase or a squash.

## Notes

- Findings are printed **redacted** in the job log, and summarised in the job summary.
- **Security tab**: with `SECURITY_TAB`, findings are uploaded as SARIF (redacted) to Security → Code scanning.
  Private repositories need GitHub Advanced Security. If the upload fails (missing permission, no code scanning),
  the job warns and still applies `FAIL_ON_LEAKS`: the scan result is never hidden by an upload problem.
  Pull requests from forks cannot upload, because their token is read-only.
- The job needs the full history (`fetch-depth: 0`); on a very large repository the checkout is the slow part.
