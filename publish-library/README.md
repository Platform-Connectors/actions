# publish-library

A reusable composite action that verifies a release tag matches the version in `pyproject.toml`, builds sdist & wheel distributions with [uv](https://github.com/astral-sh/uv), and publishes them to PyPI using [OIDC Trusted Publishing](https://docs.pypi.org/trusted-publishers/) (no API tokens required).

## Usage

```yaml
# .github/workflows/publish.yml
name: Publish Release

on:
  release:
    types: [published]

jobs:
  publish:
    name: Build & Publish to PyPI
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write   # required for OIDC trusted publishing
    steps:
      - uses: platform-connectors/actions/publish-library@v1
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `python-version` | Python version to use with uv | No | `3.11` |

### Example with inputs

```yaml
    steps:
      - uses: platform-connectors/actions/publish-library@v1
        with:
          python-version: "<python-version>"
```

## Outputs

| Name | Description |
|------|-------------|
| `version` | The project version that was published |

### Example using the output

```yaml
    steps:
      - uses: platform-connectors/actions/publish-library@v1
        id: publish

      - name: Print published version
        run: echo "Published ${{ steps.publish.outputs.version }}"
```

## Prerequisites

### 1. PyPI Trusted Publisher

Configure your PyPI project to trust this GitHub Actions workflow before running it for the first time:

1. Go to your project on [pypi.org](https://pypi.org) → **Manage** → **Publishing**.
2. Add a new **GitHub** trusted publisher with:
   - **Owner**: your GitHub org or user
   - **Repository**: your repo name
   - **Workflow name**: the filename of your calling workflow (e.g. `publish.yml`)

### 2. Version tagging convention

The action enforces that the Git tag matches the version in `pyproject.toml`. Tags may optionally be prefixed with `v`:

| Git tag | `pyproject.toml` version |
|---------|--------------------------|
| `v1.2.3` | `1.2.3` |
| `1.2.3` | `1.2.3` |

If they don't match, the workflow fails before building or publishing anything.

### 3. `pyproject.toml`

Your repository must have a `pyproject.toml` with a `[project]` version field (or dynamic version compatible with `uv version --short`).

## How it works

1. **Checkout** — checks out the repository at the tagged commit.
2. **Set up uv** — installs uv and the specified Python version.
3. **Verify tag vs version** — strips the leading `v` from the tag and compares it to the version reported by `uv version --short`. Fails fast if they differ.
4. **Build** — runs `uv build` to produce `dist/*.tar.gz` and `dist/*.whl`.
5. **Publish** — runs `uv publish dist/*` using the GitHub OIDC token for PyPI authentication.
6. **Summary** — writes the published version to the workflow run summary.
