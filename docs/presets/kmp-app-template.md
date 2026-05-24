# KMP-App-Template

JetBrains' minimal Compose Multiplatform application template. Compact (14 Kotlin files) but uses **Compose Multiplatform** (`org.jetbrains.compose.*`) — which currently exposes a mapping gap in the analyzer.

- Reference repo: [`EstebanCastel/KMP-IMPACT-KMP-App-Template-with-tool`](https://github.com/EstebanCastel/KMP-IMPACT-KMP-App-Template-with-tool)
- Baseline: [`EstebanCastel/KMP-IMPACT-KMP-App-Template-baseline`](https://github.com/EstebanCastel/KMP-IMPACT-KMP-App-Template-baseline)

## Stack

| Component | Version |
|---|---|
| AGP | 8.5 |
| Gradle wrapper | 9.3.1 |
| Kotlin | 1.9.24 |
| Compose Compiler | applied via the Compose Multiplatform plugin (Kotlin 1.9, no explicit `kotlin.plugin.compose` needed) |
| Targets | Android, iOS, Desktop |

## Known caveat — Compose Multiplatform mapping (L7)

KMP-App-Template imports Compose APIs as `androidx.compose.*` even on common source sets, but the runtime artifacts come from the `org.jetbrains.compose.*` Maven coordinates. The analyzer currently:

- Resolves the bumped Maven group `org.jetbrains.compose.material:material` to the package root `org.jetbrains.compose.material`.
- Looks for files importing `org.jetbrains.compose.material.*`.
- Finds none — because the project imports `androidx.compose.material.*`.

This is [limitation **L7**](../troubleshooting.md#l7-compose-multiplatform-mapping). The ground truth resolves the correspondence manually, so the metrics compare a *complete* GT against a *partial* automatic set; the published F1 (0.307) underestimates the true performance once the mapping is added.

If your project depends primarily on Compose Multiplatform, expect lower static metrics until the mapping ships. Recall on non-Compose dependencies (`io.ktor`, `org.jetbrains.kotlinx`, …) remains comparable to the other presets.

## Evaluation result

| Metric | Value |
|---|---|
| Kotlin files scanned | 14 |
| Direct files for `io.ktor` (ground truth) | 2 |
| Pipeline `OK` | 9 |
| Pipeline `BLOCKED_BUILD` | 1 |
| Static-analysis F1 | 0.307 (Compose Multiplatform mapping gap, L7) |
| DroidBot UTGs generated | 9/9 |

DroidBot still ran on all 9 OK PRs. The dynamic phase is not affected by the static-mapping gap.
