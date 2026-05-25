# GitHub Action

Reference for the workflow shipped under [`examples/github-action/impact-analysis.yml`](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/blob/main/examples/github-action/impact-analysis.yml).

This page documents the triggers, inputs, permissions, jobs, and artefacts. For the installation walkthrough, see [Guides → Configuring GitHub Actions](../guides/github-actions.md).

## Triggers

The workflow fires on two events:

```yaml
on:
  pull_request:
    paths:
      - "**/libs.versions.toml"
      - ".github/workflows/impact-analysis.yml"
      - "tools/kmp-impact-analyzer/**"
      - "pipeline/**"
  workflow_dispatch:
    inputs:
      dependency:        { required: true }
      before_version:    { required: true }
      after_version:     { required: true }
```

A `pull_request` event without changes to any of the four path filters is ignored by GitHub before the workflow is even queued, so the runtime cost is zero on unrelated PRs.

## `workflow_dispatch` inputs

| Input | Required | Description |
|---|---|---|
| `dependency` | yes | Dependency group (e.g. `io.ktor`). Passed straight to `--dependency` of [`analyze`](cli/analyze.md). |
| `before_version` | yes | Version before the change. |
| `after_version` | yes | Version after the change. |

All three are required by the schema. The reference workflow does **not** offer an "analyze every detected change" mode for manual runs — that responsibility belongs to the `pull_request` flow.

## Required permissions

```yaml
permissions:
  contents: write          # to push the cumulative gh-pages history branch
  pull-requests: write     # to comment on the PR with the report URL
  pages: write             # to deploy via actions/deploy-pages
  id-token: write          # required by actions/deploy-pages
```

`contents: write` is needed because the `deploy-pages` job persists the report into a cumulative history branch (`gh-pages-history`), so older PR reports survive subsequent deploys.

## Jobs

| Job | Phase | Runs on | Notes |
|---|---|---|---|
| `detect` | — | `ubuntu-latest` | Skipped for `workflow_dispatch` (the inputs are already explicit). |
| `static-pipeline` | 1 + 2 | `ubuntu-latest` | Materialises the shadow copies and runs the static analyzer. Produces `phase1/` and `phase2/`. |
| `droidbot` | 3 | `ubuntu-latest` (KVM emulator) | Builds the BEFORE and AFTER APKs, boots an Android 33 emulator, runs DroidBot on each APK. |
| `merge` | 4 | `ubuntu-latest` | Consolidates the static and dynamic artefacts into `phase4/consolidated.json`. Builds the HTML report. |
| `deploy-pages` | 5 | `ubuntu-latest` | Uploads the report bundle and deploys it through `actions/deploy-pages@v4`. Posts the PR comment. |

`static-pipeline` and `droidbot` run **in parallel**. Total wall-clock is bounded by the slower of the two (DroidBot, ~8–11 min).

## `detect` job outputs

When triggered by `pull_request`, `detect` exposes the parsed bump for downstream jobs:

| Output | Type | Notes |
|---|---|---|
| `has_changes` | `"true" \| "false"` | Whether the catalog diff contained at least one entry. |
| `dependency_group` | `str` | Maven group of the first detected change. |
| `before_version` | `str` | BEFORE version of the first change. |
| `after_version` | `str` | AFTER version of the first change. |

The downstream jobs gate themselves on `needs.detect.outputs.has_changes == 'true'`, so PRs that don't touch the catalog finish in under 30 seconds.

## Android module autodetection

The workflow probes for the KMP Android module by name, in this order:

```text
shared, composeApp, androidApp, app, common, kmm-shared, kmpShared
```

If none of these exist, it falls back to `:app`. The detected module name is written into `$GITHUB_OUTPUT` as `shared_module` and reused throughout the workflow.

If your project does not match any of these conventions, the cleanest fix is to add a Gradle module alias — for example, expose your real module under `composeApp`. Patching the workflow itself works but loses the upstream upgrade path.

## DroidBot configuration

The DroidBot invocation is baked into the workflow:

```bash
droidbot -a /tmp/before.apk -o /tmp/droidbot-before \
  -policy dfs_greedy -count 100 -timeout 90 \
  -grant_perm -is_emulator
```

To change the budget (`-count`) or the policy (`-policy`), edit the workflow itself — these are not exposed as inputs. The bundled CLI does honour `AnalysisConfig.droidbot_timeout` and `droidbot_policy`, but the reference workflow drives DroidBot directly to keep the static and dynamic jobs decoupled.

## Workflow artefacts

| Artefact | Source | Retained for |
|---|---|---|
| `static-artifacts` | `output/phase1/**`, `output/phase2/**` | Workflow defaults (90 days). |
| `droidbot-utg` | `output/phase3/**` | Workflow defaults. |
| `consolidated` | `output/phase4/consolidated.json` | Workflow defaults. |
| `report-bundle` | `output/report/**`, `output/phase5/**` | Used by `deploy-pages`. |

The bundle published to Pages is *cumulative* — each PR's report lives under a stable URL of the form
`https://<owner>.github.io/<repo>/reports/<dependency>/<before>-to-<after>/run-<runId>/`.

## See also

- [Guides → Configuring GitHub Actions](../guides/github-actions.md) — installation walkthrough.
- [Guides → Configuring Dependabot](../guides/dependabot.md) — biasing PRs toward bumps that keep the build healthy.
- [`detect-version-changes`](cli/detect-version-changes.md) — the CLI command invoked by the `detect` job.
