# Confetti

A conference-companion Kotlin Multiplatform app targeting Android, iOS, JVM, and a Compose-for-Web frontend. Largest tree in the evaluation (299 Kotlin files).

- Reference repo: [`EstebanCastel/KMP-IMPACT-Confetti-with-tool`](https://github.com/EstebanCastel/KMP-IMPACT-Confetti-with-tool)
- Baseline: [`EstebanCastel/KMP-IMPACT-Confetti-baseline`](https://github.com/EstebanCastel/KMP-IMPACT-Confetti-baseline)

## Stack

| Component | Version |
|---|---|
| AGP | 8.13 |
| Gradle wrapper | 9.3.1 |
| Kotlin | 2.2.21 |
| Compose Compiler plugin | applied (Kotlin 2.x) |
| Targets | Android, iOS, JVM, Wear, Web |

Confetti uses **Gradle 9.x** with **AGP 8.x**. The pipeline supports this combination; the Gradle 9 line is required for projects that pin AGP 8.13 or later.

## Upstream workflows — disable before adopting

The upstream `joreilly/Confetti` repository ships 12 workflows that depend on credentials you almost certainly do not have in your fork (Google Cloud, Firebase, App Store Connect, Android signing keystore). Leaving them enabled makes every PR look red even when KMP-IMPACT runs cleanly.

Disable them once per repository:

```bash
gh api repos/<owner>/<repo>/actions/workflows --jq \
  '.workflows[] | select(.path | startswith(".github/workflows/")) |
   select(.path != ".github/workflows/impact-analysis.yml") | .id' \
| while read wid; do
    gh api -X PUT repos/<owner>/<repo>/actions/workflows/$wid/disable
  done
```

Workflows you typically disable in Confetti:

```text
android_wear_jvm.yml         publish-android.yml
backend-deploy.yml           publish-kmmbridge.yml
backend-test.yml             verify.yml
build-and-publish-web.yml    fixes.yml
ios.yml                      junie.yml
preview-baselines.yml        preview-comment.yml
```

## Dependabot biasing

Same conservative biasing as [Pokedex](pokedex.md), plus an explicit ignore for the Wear workflow's dependencies if you have disabled the Wear job:

```yaml
- dependency-name: "androidx.wear*"
  update-types: ["version-update:semver-major", "version-update:semver-minor"]
```

## Scale notes

Confetti is the largest reference tree (299 Kotlin files). The static phase scales linearly with file count and completes in under 90 seconds in CI. Pages deploys can hit the concurrency cap [L6](../troubleshooting.md#l6-deploy-pages-cancellation) when several PRs land at once — the workflow already sets the recommended concurrency group.
