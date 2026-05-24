# GitHub Action

The recommended way to run KMP-IMPACT in production is as a GitHub Actions workflow that fires on every PR that touches `gradle/libs.versions.toml`. The workflow:

1. checks out BEFORE (the PR's merge-base) and AFTER (the PR head),
2. builds the Android APK on both states,
3. runs the full pipeline,
4. publishes the report to GitHub Pages,
5. comments on the PR with the URL and a short risk summary.

A complete copy-paste workflow lives at [`examples/github-action/impact-analysis.yml`](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/blob/main/examples/github-action/impact-analysis.yml) in the repository.

## Installation in a target repo

Three files to copy:

```text
.github/
├── workflows/
│   └── impact-analysis.yml      # the pipeline
└── dependabot.yml               # auto-open PRs for libs.versions.toml bumps
tools/
└── kmp-impact-analyzer/         # vendored copy of the tool (or git submodule)
```

Then enable GitHub Pages for the repo with **Source: GitHub Actions**:

```bash
gh api -X PUT "repos/$OWNER/$REPO/pages" \
  -f "build_type=workflow"
```

## Recommended Dependabot configuration

Bias Dependabot toward minor/patch bumps so the pipeline always has a compilable AFTER APK. Major bumps of foundational tooling (Kotlin, Compose, Coroutines, Serialization, Ktor, AGP) often introduce ABI breaks that block APK assembly — when that happens DroidBot has no AFTER state to explore and the dynamic phase degrades.

```yaml title=".github/dependabot.yml"
--8<-- "examples/github-action/dependabot.yml"
```

## How the PR comment looks

```text
KMP-IMPACT analysis · io.ktor 2.3.8 -> 2.3.11

Static impact      : 7 files (2 direct, 5 transitive)
Source sets        : commonMain(5), androidMain(1), iosMain(1)
expect/actual      : 1 pair touched
Dynamic impact     : 0 new screens, 0 missing screens
Risk label         : LOW
Recommendation     : safe to merge after CI

Full report   -> https://<owner>.github.io/<repo>/pr-42/
Raw artifacts -> workflow run #123
```

The risk label is a review-prioritization cue, not a validated failure predictor; see [Architecture → Phase contracts](../architecture/phases.md).

## Concurrency and Pages cancellations

`actions/deploy-pages@v4` enforces concurrency at the repository level. If multiple PRs of the same repo finish at the same time, later deploys cancel earlier ones. The reference workflow already sets:

```yaml
concurrency:
  group: pages-${{ github.ref }}
  cancel-in-progress: false
```

But the same conflict can appear across repos owned by the same user when running large evaluations in parallel. The [`evaluacion/scripts/serial_rerun.sh`](https://github.com/EstebanCastel/KMP-IMPACT-Pokedex-with-tool/blob/main/evaluacion/scripts/serial_rerun.sh) helper used in the thesis re-runs cancelled deploys serially with `gh run watch`.

## What if the APK won't build?

The workflow uses `|| exit 1` on every `./gradlew ... assembleDebug` step so a build failure is **explicit**, not silently swallowed. When that happens the report still publishes, with the dynamic tab showing `BLOCKED — APK assembly failed` and a link to the Gradle log.

See [Troubleshooting → L4](../troubleshooting.md#l4-droidbot-blocked-no-apk) for the full triage flow.
