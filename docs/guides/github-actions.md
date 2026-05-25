# Configuring GitHub Actions

Wire KMP-IMPACT into a target repository so the pipeline runs on every pull request that touches `gradle/libs.versions.toml`. The workflow checks out BEFORE and AFTER, builds the APKs, runs the full pipeline, publishes the HTML report to GitHub Pages, and comments on the PR.

## Files to install

Drop two files into the target repo:

```text
.github/
├── workflows/
│   └── impact-analysis.yml      # the pipeline
└── dependabot.yml               # auto-open PRs when versions change
tools/
└── kmp-impact-analyzer/         # vendored copy of the analyzer
```

A complete copy-paste workflow lives at [`examples/github-action/impact-analysis.yml`](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/tree/main/examples/github-action).

## Installation in three steps

### 1. Copy the workflow and Dependabot config

```bash
mkdir -p .github/workflows
cp /path/to/kmp-impact/examples/github-action/impact-analysis.yml .github/workflows/
cp /path/to/kmp-impact/examples/github-action/dependabot.yml      .github/
```

### 2. Vendor the analyzer

```bash
git clone --depth 1 \
  https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform.git \
  tools/kmp-impact-analyzer
rm -rf tools/kmp-impact-analyzer/.git
```

Alternatively keep it as a Git submodule for easier upgrades.

### 3. Enable GitHub Pages with `Source: GitHub Actions`

```bash
gh api -X PUT "repos/<owner>/<repo>/pages" -f "build_type=workflow"
```

Verify Actions permissions in `Settings → Actions → General → Workflow permissions`:

| Permission | Value |
|---|---|
| `contents` | read |
| `pull-requests` | write |
| `pages` | write |
| `id-token` | write |

Push the changes. On the next Dependabot PR, the workflow runs end-to-end.

## What the workflow does

The workflow has five jobs that mirror the pipeline phases:

| Job | Phase | Action |
|---|---|---|
| `detect` | — | Parses the PR diff against `gradle/libs.versions.toml`. Skips the rest if nothing relevant changed. |
| `static-pipeline` | 1 + 2 | Runs the static phase against the BEFORE (`merge-base`) and AFTER (PR head) shadow copies. |
| `droidbot` | 3 | Boots an Android emulator on `ubuntu-latest`, builds BEFORE and AFTER APKs, runs DroidBot against each. |
| `merge` | 4 | Combines the static and dynamic artifacts into the consolidated JSON. |
| `deploy-pages` | 5 | Bundles the HTML report and publishes it via `actions/deploy-pages@v4`. |

If any step legitimately blocks (no APK, no UTG, catalog not touched), the workflow surfaces the reason — see [Diagnosing a BLOCKED phase](diagnosing-blocked.md).

## Customization knobs

### Android module name

The workflow tries `:android`, `:app`, `:composeApp`, `:androidApp`, `:android-app`, `:shared:android`, and `:app:android` in order. Override with an environment variable:

```yaml
env:
  KMP_IMPACT_ANDROID_MODULE: ":your-android-module"
```

### Gradle wrapper

If the project's wrapper is older than the AGP requires (8.7 for AGP 8.x, 9.0 for AGP 9.x), the workflow upgrades it in place for the duration of the run. No commit is created.

### DroidBot exploration budget

Defaults are `count = 100`, `timeout = 90`, `policy = dfs_greedy`. Increase for larger apps; the runtime cost scales roughly linearly.

## Concurrency and Pages cancellations

`actions/deploy-pages@v4` enforces concurrency at the repository level — one active deployment at a time. The reference workflow sets:

```yaml
concurrency:
  group: pages-${{ github.ref }}
  cancel-in-progress: false
```

This bounds the conflict to *across-PR* contention. For multi-PR bursts, re-run cancelled deploys serially with `gh run watch` between calls. See [L6 in Troubleshooting](../troubleshooting.md#l6-deploy-pages-cancellation).

## Next steps

- [Configuring Dependabot](dependabot.md) — bias toward minor/patch bumps so the AFTER APK keeps compiling.
- [Reading the Report](reading-the-report.md) — what each tab in the HTML output shows.
- [Reference → GitHub Action](../reference/github-action.md) — inputs, outputs, and the full env-var reference.
