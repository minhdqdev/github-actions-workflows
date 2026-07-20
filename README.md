# github-actions-workflows

Org-wide reusable GitHub Actions workflows for `minhdqdev-org`.

## Available Workflows

### `ci.yml` — Docker CI

Builds and pushes a Docker image to Harbor, then sends a success notification to NotiHub.
Branch is detected at runtime to apply the correct tag strategy:

| Branch | Tag produced | Environment | ArgoCD strategy |
|--------|-------------|-------------|-----------------|
| `develop` | `dev-<short_sha>` | DEV | `latest`, `regexp:^dev-.*$` |
| `rc/<version>` | `<version>-rc.<commit_count>` | UAT | `semver`, `regexp:^.*-rc\.[0-9]+$` |
| `main` / `master` | `<version>` + `latest` | PROD | `semver` |

Version is resolved from `package.json`, `pyproject.toml`, or `Cargo.toml`. Falls back to
the short commit SHA if none are present.

**Name RC branches after the version they release** — `rc/0.1.120`, not `rc/my-feature`. The
workflow triggers on `rc/**` so any name builds, but the branch name is how anyone reading the
branch list knows which release is in UAT.

`<commit_count>` is `git rev-list --count HEAD`, **not** `github.run_number`: it is per-branch
monotonic with no gaps from unrelated branches. This is why the checkout step sets
`fetch-depth: 0` — under the default shallow clone the count is always `1`, every RC builds
`<version>-rc.1`, and ArgoCD Image Updater sees no newer tag and silently stops deploying.

It also means **the version must be bumped for each new RC of the same release**. Two RC builds at
the same version and commit count collide on the tag; the second overwrites the first and nothing
deploys.

#### Usage

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [develop, main, "rc/**"]

jobs:
  ci:
    uses: minhdqdev-org/github-actions-workflows/.github/workflows/ci.yml@main
```

#### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `dockerfile` | `./Dockerfile` | Path to the Dockerfile |
| `dockerfile_target` | `""` | Build target for multi-stage Dockerfiles |

#### Custom Dockerfile path

```yaml
jobs:
  ci:
    uses: minhdqdev-org/github-actions-workflows/.github/workflows/ci.yml@main
    with:
      dockerfile: ./deploy/dockerfile
```

#### Multi-stage build target

```yaml
jobs:
  ci:
    uses: minhdqdev-org/github-actions-workflows/.github/workflows/ci.yml@main
    with:
      dockerfile_target: production
```

#### Test prerequisite before Docker build

```yaml
jobs:
  test:
    if: github.ref == 'refs/heads/main'
    runs-on: [self-hosted, minhdqdev-org, manual]
    steps:
      - uses: actions/checkout@v4
      - run: bun install --frozen-lockfile
      - run: bun test --coverage

  ci:
    needs: [test]
    if: always() && (needs.test.result == 'success' || needs.test.result == 'skipped')
    uses: minhdqdev-org/github-actions-workflows/.github/workflows/ci.yml@main
```

## Requirements

- **Runner**: `[self-hosted, minhdqdev-org, manual]` with Harbor credentials injected via
  `envFrom` (`HARBOR_USERNAME`, `HARBOR_PASSWORD`) on the runner pod.
- **Registry**: `harbor.internal.minhdq.dev/minhdqdev/<repo-name>`
- **Build cache**: stored at `harbor.internal.minhdq.dev/minhdqdev/<repo-name>:buildcache`

## Further reading

- [Shared CI Workflows standard](https://github.com/minhdqdev-org/engineering-docs/blob/main/docs/design-guidelines/shared-ci-workflows.md)
- [ArgoCD Image Updater standard](https://github.com/minhdqdev-org/engineering-docs/blob/main/docs/design-guidelines/argocd-image-updater.md)
