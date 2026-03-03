# Container Build & Publish Action

[![Build](https://img.shields.io/github/actions/workflow/status/netwarlan/action-container-build/version.yaml)](https://github.com/netwarlan/action-container-build/actions)
[![Release](https://img.shields.io/github/v/release/netwarlan/action-container-build)](https://github.com/netwarlan/action-container-build/releases)
[![Discord](https://img.shields.io/badge/Discord-Join-5865F2?logo=discord&logoColor=white)](https://discord.gg/QtqKW9xvzK)

A reusable GitHub Actions workflow for building and publishing Docker container images to GitHub Container Registry (GHCR).

## Features

- Builds and publishes Docker images to `ghcr.io/netwarlan`
- Automatic tagging based on branch (`main` → `latest`, others → branch name)
- Multi-platform build support (amd64, arm64, etc.)
- GitHub Actions cache for faster builds
- OCI image labels for traceability (git sha, repo URL, etc.)
- Concurrency control to prevent duplicate builds
- Dependabot configured for automatic action updates

## Usage

### Basic Usage

```yaml
name: Build Container

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    uses: netwarlan/action-container-build/.github/workflows/build-publish.yaml@main
    with:
      image-name: my-application
```

### Advanced Usage

```yaml
jobs:
  build:
    uses: netwarlan/action-container-build/.github/workflows/build-publish.yaml@main
    with:
      image-name: my-application
      dockerfile: ./docker/Dockerfile.prod
      context: ./docker
      platforms: linux/amd64,linux/arm64
```

### Using Outputs

```yaml
jobs:
  build:
    uses: netwarlan/action-container-build/.github/workflows/build-publish.yaml@main
    with:
      image-name: my-application

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy image
        run: |
          echo "Deploying ${{ needs.build.outputs.image }}@${{ needs.build.outputs.digest }}"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `image-name` | Name of the image to build and deploy | Yes | - |
| `dockerfile` | Path to Dockerfile | No | `./Dockerfile` |
| `context` | Build context path | No | `.` |
| `platforms` | Target platforms (comma-separated) | No | `linux/amd64` |
| `version` | Semantic version tag to apply to the image (e.g., `v1.2.3`) | No | `''` |
| `build-args` | Newline-separated list of build-time variables | No | `''` |
| `push` | Whether to push the image to the registry | No | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `digest` | Image digest (sha256) |
| `image` | Full image reference (without tag) |

## Image Tags

Images are automatically tagged based on the branch:

| Branch | Tags |
|--------|------|
| `main` | `latest`, `<branch>`, `<sha>` |
| Other | `<branch>`, `<sha>` |

When `version` is provided, the version tag (e.g., `v1.2.3`) is also applied.

## Requirements

The calling repository must have:

- A `Dockerfile` (or specify path via `dockerfile` input)
- Write access to the `ghcr.io/netwarlan` registry (automatic for repos in the netwarlan org)

## License

MIT
