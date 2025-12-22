# github-actions-workflows

This repository contains a collection of reusable GitHub Actions workflows.

## Available Workflows

### Django CI (`ci/django.yml`)

Builds a Django application, extracts version from `pyproject.toml`, and pushes a Docker image to Harbor.

**Usage:**

```yaml
jobs:
  ci:
    uses: minhdqdev/github-actions-workflows/ci/django.yml@main
    with:
      image_name: my-django-app
      # Optional inputs:
      # python-version: "3.11"
      # dockerfile: "./deploy/dockerfile"
    secrets:
      HARBOR_USERNAME: ${{ secrets.HARBOR_USERNAME }}
      HARBOR_PASSWORD: ${{ secrets.HARBOR_PASSWORD }}
```

### Next.js CI (`ci/nextjs.yml`)

Installs dependencies, lints, builds a Next.js application, extracts version from `package.json`, and pushes a Docker image to Harbor.

**Usage:**

```yaml
jobs:
  ci:
    uses: minhdqdev/github-actions-workflows/ci/nextjs.yml@main
    with:
      image_name: my-nextjs-app
      # Optional inputs:
      # node-version: "20"
      # dockerfile: "./Dockerfile"
    secrets:
      HARBOR_USERNAME: ${{ secrets.HARBOR_USERNAME }}
      HARBOR_PASSWORD: ${{ secrets.HARBOR_PASSWORD }}
```

### Generic Docker CI (`ci/docker.yml`)

A generic workflow for building and pushing Docker images for any project type.

**Usage:**

```yaml
jobs:
  ci:
    uses: minhdqdev/github-actions-workflows/ci/docker.yml@main
    with:
      image_name: my-app
      # Optional inputs:
      # dockerfile: "./Dockerfile"
      # version: "1.0.0"
    secrets:
      HARBOR_USERNAME: ${{ secrets.HARBOR_USERNAME }}
      HARBOR_PASSWORD: ${{ secrets.HARBOR_PASSWORD }}
```
