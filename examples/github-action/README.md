# `examples/github-action`

Drop-in GitHub Actions integration for KMP-IMPACT. This directory contains the two files that make the analyzer run automatically on every Dependabot pull request that touches `gradle/libs.versions.toml`.

```text
examples/github-action/
├── impact-analysis.yml   # the pipeline workflow
└── dependabot.yml        # Dependabot configuration biased toward minor/patch bumps
```

## Installation in a target repository

1. Copy both files into the target repo:

   ```bash
   mkdir -p .github/workflows
   cp /path/to/kmp-impact/examples/github-action/impact-analysis.yml .github/workflows/
   cp /path/to/kmp-impact/examples/github-action/dependabot.yml      .github/
   ```

2. Vendor the analyzer under `tools/kmp-impact-analyzer/` (the workflow expects it at this path):

   ```bash
   git clone --depth 1 https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform.git \
     tools/kmp-impact-analyzer
   rm -rf tools/kmp-impact-analyzer/.git
   ```

   Alternatively keep it as a Git submodule for easier upgrades.

3. Enable GitHub Pages with `Source: GitHub Actions`:

   ```bash
   gh api -X PUT "repos/<owner>/<repo>/pages" -f "build_type=workflow"
   ```

4. Verify Actions permissions on the repo: `contents: read`, `pull-requests: write`, `pages: write`, `id-token: write`.

5. Commit and push. The next time Dependabot opens a PR, the workflow runs end-to-end.

## What the workflow does

The workflow has five jobs that mirror the pipeline phases:

1. **`detect`** — parses the PR diff against `gradle/libs.versions.toml` and decides whether to continue.
2. **`static-pipeline`** — runs the static phase against the BEFORE (merge-base) and AFTER (PR head) shadow copies.
3. **`droidbot`** — boots an Android emulator on `ubuntu-latest`, builds the BEFORE and AFTER APKs, and runs DroidBot against each.
4. **`merge`** — combines the static and dynamic artifacts and produces the final consolidated JSON.
5. **`deploy-pages`** — bundles the HTML report and publishes it via `actions/deploy-pages@v4`. Comments on the PR with the URL.

If any step legitimately blocks (no APK produced, no UTG generated, version catalog not touched), the workflow surfaces the reason rather than masking it as `skipped`. See the pipeline reference in [`docs/architecture/`](../../docs/architecture/) and the data contracts in [`docs/reference/contracts/`](../../docs/reference/contracts/).

## Customization

- **Android module name.** The workflow tries `:android`, `:app`, `:composeApp`, `:androidApp`, `:android-app`, `:shared:android`, and `:app:android` in order. If your module has a different name, set:

  ```yaml
  env:
    KMP_IMPACT_ANDROID_MODULE: ":your-android-module"
  ```

- **JDK / Gradle / Kotlin versions.** The workflow defaults to JDK 21 and uses the project's Gradle wrapper. If the wrapper is older than the AGP requires (8.7 for AGP 8.x, 9.0 for AGP 9.x), the workflow upgrades it in-place for the duration of the run.

- **DroidBot exploration budget.** Defaults are `count = 100`, `timeout = 90`, `policy = dfs_greedy`. Increase for larger applications; the runtime cost scales roughly linearly.

- **Dependabot biasing.** Edit `dependabot.yml` to match your project's policy. The shipped defaults block major bumps of Kotlin, Compose, Coroutines, Serialization, Ktor, AGP, and KSP — major bumps of these tend to introduce ABI breaks that prevent the AFTER APK from building.

## Troubleshooting

When a run does not produce the report you expect, walk the diagnosis flow in [`docs/troubleshooting.md`](../../docs/troubleshooting.md). The most common cases:

| Symptom | First check |
|---|---|
| Workflow says `no Gradle catalog change found` | The PR does not touch `gradle/libs.versions.toml`. **L1** — direct `build.gradle.kts` edits are not detected. |
| `Build BEFORE APK` fails | Gradle wrapper version vs. AGP version. **L4** triage. |
| `DroidBot produced no UTG artifact` | Emulator boot, APK install, or `adb` permissions. **L4** triage. |
| `deploy-pages` cancelled | Concurrent deploys across PRs. **L6**. |
| UI shows many red checks but `impact-analysis.yml` is green | Upstream workflows requiring missing secrets. Disable them in bulk (see the [Troubleshooting page](../../docs/troubleshooting.md)). |
