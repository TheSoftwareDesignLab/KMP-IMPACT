# GitHub Action

Reference for the workflow shipped under [`examples/github-action/`](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/tree/main/examples/github-action).

This page documents the inputs, outputs, environment variables, and job structure. For the how-to, see [Guides → Configuring GitHub Actions](../guides/github-actions.md).

## Triggers

```yaml
on:
  pull_request:
    paths:
      - "gradle/libs.versions.toml"
  workflow_dispatch:
    inputs:
      dependency:        { required: false }
      before_version:    { required: false }
      after_version:     { required: false }
```

The workflow runs on every pull request that touches `gradle/libs.versions.toml`, and can be invoked manually with explicit inputs for ad-hoc analyses.

## Inputs (workflow_dispatch)

| Input | Required | Description |
|---|---|---|
| `dependency` | no | Maven group or coordinate substring. If absent, the workflow runs against every change detected in the catalog diff. |
| `before_version` | no | Used together with `dependency` to override automatic detection. |
| `after_version` | no | Used together with `dependency` to override automatic detection. |

## Environment variables

| Variable | Purpose | Default |
|---|---|---|
| `KMP_IMPACT_ANDROID_MODULE` | Gradle path to the Android application module. | Auto-detected — tries `:android`, `:app`, `:composeApp`, `:androidApp`, `:android-app`, `:shared:android`, `:app:android`. |
| `KMP_IMPACT_DROIDBOT_COUNT` | DroidBot event budget. | `100` |
| `KMP_IMPACT_DROIDBOT_TIMEOUT` | DroidBot per-event timeout in seconds. | `90` |
| `KMP_IMPACT_DROIDBOT_POLICY` | DroidBot exploration policy. | `dfs_greedy` |

## Job structure

| Job | Phase | Output |
|---|---|---|
| `detect` | — | `outputs.has-changes`, parsed catalog diff. |
| `static-pipeline` | 1 + 2 | `phase1/`, `phase2/` uploaded as the `static-artifacts` artifact. |
| `droidbot` | 3 | `phase3/` uploaded as the `droidbot-utg` artifact. |
| `merge` | 4 | `phase4/consolidated.json`. |
| `deploy-pages` | 5 | The HTML report on GitHub Pages. PR comment posted with the URL. |

## Required permissions

```yaml
permissions:
  contents: read
  pull-requests: write
  pages: write
  id-token: write
```

## Outputs uploaded as workflow artifacts

| Artifact | Source |
|---|---|
| `static-artifacts` | `output/phase{1,2}/**` |
| `droidbot-utg` | `output/phase3/**` |
| `consolidated` | `output/phase4/consolidated.json` |
| `report-bundle` | `output/report/**` and `output/phase5/**` |

## See also

- [Guides → Configuring GitHub Actions](../guides/github-actions.md) — installation walkthrough.
- [Guides → Configuring Dependabot](../guides/dependabot.md) — biasing PRs toward bumps that keep the build healthy.
- [`detect-version-changes`](cli/detect-version-changes.md) — the CLI command invoked by the `detect` job.
