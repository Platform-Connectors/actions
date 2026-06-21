# ci

Reusable composite action for Python quality checks using [uv](https://github.com/astral-sh/uv).

It runs:

1. Ruff
2. MyPy
3. Pylint
4. Pytest with coverage
5. Diff coverage on pull requests

## Usage

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  checks:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: Platform-Connectors/actions/ci@v1
        id: ci
        with:
          python-version: "<python-version>"
          package-source-folder: <package-folder>

      - name: Coverage summary from action outputs
        run: |
          echo "Total coverage: ${{ steps.ci.outputs.total-coverage }}%"
          echo "Diff coverage: ${{ steps.ci.outputs.diff-coverage }}%"
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `python-version` | Python version used by the action | No | `3.11` |
| `package-source-folder` | Package folder for pylint/pytest coverage target (example: `src/gallagher_restapi`) | Yes | n/a |
| `run-ruff` | Whether to run Ruff | No | `true` |
| `run-mypy` | Whether to run MyPy | No | `true` |
| `run-pylint` | Whether to run Pylint | No | `true` |
| `run-pytest` | Whether to run Pytest + coverage | No | `true` |
| `total-coverage-threshold` | Minimum overall codebase coverage percentage enforced | No | `95` |
| `diff-coverage-threshold` | Minimum changed-lines coverage percentage enforced by diff-cover | No | `100` |

## Outputs

| Name | Description |
|------|-------------|
| `coverage-report` | Coverage XML file path (`coverage.xml`) |
| `total-coverage` | Total line coverage percentage from coverage.xml |
| `diff-coverage` | Changed-lines coverage percentage for pull requests |

## Coverage Reporting Recommendation

Use the action outputs as the contract between this action and the calling workflow:

1. Use `steps.<id>.outputs.total-coverage` and `steps.<id>.outputs.diff-coverage` for workflow-level conditions, notifications, or PR comments.
2. Keep the human-readable details in `GITHUB_STEP_SUMMARY` (already written by this action).
3. If your workflow needs historical trend data, upload `steps.<id>.outputs.coverage-report` as an artifact in the caller workflow.

Example artifact upload in caller workflow:

```yaml
      - name: Upload coverage.xml
        uses: actions/upload-artifact@v4
        with:
          name: coverage-xml
          path: ${{ steps.ci.outputs.coverage-report }}
```
