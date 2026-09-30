# build-docker.yml

Builds a container image with [buildx](https://docs.docker.com/build/) and pushes it to a registry, GitHub
Container Registry by default.

```yaml
build:
  needs: release
  if: needs.release.outputs.release-created == 'true'
  uses: Mitchou10/github-workflow/.github/workflows/build-docker.yml@v0
  permissions:
    contents: read
    packages: write
  with:
    IMAGE_TAG: ${{ needs.release.outputs.version }}
    LATEST_TAG: ${{ github.ref_name == 'main' }}
```

A complete pipeline (release, build, scan) is in [examples/docker/caller.yml](../examples/docker/caller.yml).

## Permissions

The workflow declares none: it gets those of the calling job.

| Use | The calling job needs |
| --- | --- |
| Build only (`PUSH: false`) | `contents: read` |
| Push to `ghcr.io` | `contents: read` and `packages: write` |
| Push to another registry | `contents: read`, plus `REGISTRY_USERNAME` / `REGISTRY_PASSWORD` |

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `IMAGE_NAME` | empty | Image without tag. Empty means `ghcr.io/<owner>/<repository>`, lower-cased |
| `IMAGE_TAG` | empty | Main tag, typically the release version. Empty means `sha-<short sha>` |
| `LATEST_TAG` | `false` | Also tag `latest` |
| `TAG_MAJOR_AND_MINOR` | `false` | Also tag `X.Y` and `X`. Skipped for prereleases such as `1.2.3-rc.1` |
| `TAG_SHORT_SHA` | `false` | Also tag `sha-<short sha>` |
| `CONTEXT` | `.` | Build context |
| `DOCKERFILE` | empty | Dockerfile path. Empty means `<CONTEXT>/Dockerfile` |
| `TARGET` | empty | Stage of a multi-stage Dockerfile |
| `PLATFORMS` | `linux/amd64` | Comma-separated, for example `linux/amd64,linux/arm64` |
| `BUILD_ARGS` | empty | `KEY=value`, one per line |
| `PUSH` | `true` | Push. `false` only checks that the image builds |
| `CACHE` | `true` | Layer cache in the GitHub Actions cache |
| `CACHE_MODE` | `min` | `min` caches the final layers, `max` all of them |
| `PROVENANCE` | `false` | Provenance attestation |
| `RUNS_ON` | `["ubuntu-24.04"]` | Runner labels, as a JSON array |

Secrets, all optional: `REGISTRY_USERNAME` and `REGISTRY_PASSWORD` (default: the GitHub actor and the job token,
which is right for `ghcr.io`), and `BUILD_SECRETS` (`id=value` per line, read in the Dockerfile with
`RUN --mount=type=secret,id=...`, never stored in a layer).

Outputs: `image` (name without tag), `digest` (`sha256:...`), `tags` (one per line).

## Tags

| Tag | When |
| --- | --- |
| `IMAGE_TAG` | Always |
| `latest` | `LATEST_TAG` |
| `X.Y`, `X` | `TAG_MAJOR_AND_MINOR` and `IMAGE_TAG` is a plain `X.Y.Z` (a leading `v` is accepted) |
| `sha-abc1234` | `TAG_SHORT_SHA` |

On the two-branch flow, release candidates should not move `latest` or the floating tags. The example uses
`github.ref_name == 'main'` for both.

The image also gets the standard OCI labels (source repository, revision, version, creation date).

## Several images

Use a matrix in the caller, with one `IMAGE_NAME` and `CONTEXT` per entry. Each image has its own layer cache
(the cache scope comes from the image name), so builds do not evict each other.

## Multi-platform

`PLATFORMS: linux/amd64,linux/arm64` builds both in one job. QEMU is set up automatically as soon as a platform
other than amd64 is asked for, and it is **slow** (a build several times longer than native). For frequent builds
of an arm64 image, a native arm runner (`RUNS_ON: '["ubuntu-24.04-arm"]'` with a single platform) is faster, at
the price of merging the manifests yourself.

## Notes

- `PROVENANCE` is off because buildx otherwise pushes an extra untagged manifest, which shows up as an
  `unknown/unknown` platform in the ghcr.io package page.
- `CACHE_MODE: max` speeds up multi-stage builds but fills the 10 GB cache of the repository quickly.
- The workflow builds; scan the result afterwards with [scan-trivy](scan-trivy.md) (`SCAN_TYPE: image`).
- A first push to `ghcr.io` creates a private package linked to the repository. Make it public in the package
  settings if the image must be pulled anonymously.
