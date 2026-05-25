# Presets

Reference integrations for the five Kotlin Multiplatform projects used to validate KMP-IMPACT. Each preset documents the exact stack (AGP, Gradle, Kotlin), the catalog layout, the Android module to target, the Dependabot biasing applied, and the known caveats for that project's structure.

Use a preset as a *checklist*: if your project matches one of these stacks, the same configuration works without modification.

| Preset | Stack | Notes |
|---|---|---|
| [Pokedex](pokedex.md) | AGP 8.5 + Gradle 8.10.2 + Kotlin 2.0.21 | Compose Multiplatform on Android + Desktop. Kotlin 2.x — Compose Compiler plugin mandatory. |
| [Confetti](confetti.md) | AGP 8.13 + Gradle 9.3.1 + Kotlin 2.2.21 | Largest reference tree (299 Kotlin files). Has heavy upstream workflows that must be disabled before adoption. |
| [DroidconKotlin](droidcon-kotlin.md) | Gradle 8.14.3 + Kotlin 2.2.21 | Conventional KMP layout with clean Maven-to-Kotlin mapping. Has 2 upstream workflows to disable. |
| [KMP-App-Template](kmp-app-template.md) | AGP 8.5 + Gradle 9.3.1 + Kotlin 1.9.24 | Compose Multiplatform (`org.jetbrains.compose.*`) — exposes the L7 mapping gap. |
| [kmp-production-sample](kmp-production-sample.md) | AGP 8.1.4 + Gradle 8.5 + Kotlin 1.9.21 | Multi-target (Android, iOS, JVM, JS). Useful as a no-Compose control. |

## How to adapt a preset to your repository

1. Pick the preset whose Android module name and source-set layout most closely matches yours.
2. Copy `examples/github-action/impact-analysis.yml` to `.github/workflows/impact-analysis.yml` in the target repo.
3. Copy the preset's `dependabot.yml` to `.github/dependabot.yml`, then adjust the `ignore` rules to your project's policy.
4. If your Android module is not detected automatically (see [Architecture](../architecture/pipeline-overview.md)), set the env var `KMP_IMPACT_ANDROID_MODULE=":your-module"` in the workflow.
5. Enable GitHub Pages with **Source: GitHub Actions**.
6. Disable any upstream workflow that requires secrets you do not have (see the Confetti and DroidconKotlin presets for the disable command).
