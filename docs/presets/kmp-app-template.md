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

This is [limitation **L7**](../troubleshooting.md#l7-compose-multiplatform-mapping). If your project depends primarily on Compose Multiplatform, expect the static phase to under-report impact until the mapping ships. The dynamic phase is not affected.
