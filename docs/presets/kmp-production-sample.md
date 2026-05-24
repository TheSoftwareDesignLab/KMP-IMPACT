# kmp-production-sample

A Kotlin Multiplatform RSS reader targeting Android, iOS, JVM, and JS. Multi-target, conventional source-set layout, older AGP.

- Reference repo: [`EstebanCastel/KMP-IMPACT-kmp-production-sample-with-tool`](https://github.com/EstebanCastel/KMP-IMPACT-kmp-production-sample-with-tool)
- Baseline: [`EstebanCastel/KMP-IMPACT-kmp-production-sample-baseline`](https://github.com/EstebanCastel/KMP-IMPACT-kmp-production-sample-baseline)

## Stack

| Component | Version |
|---|---|
| AGP | 8.1.4 |
| Gradle wrapper | 8.5 |
| Kotlin | 1.9.21 |
| Compose Compiler | not used (no Compose) |
| Targets | Android, iOS, JVM, JS |

This preset is the only one in the evaluation that does **not** use Compose. Useful as a control: ABI-breaking issues from Compose Compiler or BOM bumps do not apply here.

## Dependabot biasing

```yaml
- package-ecosystem: "gradle"
  directory: "/"
  schedule: { interval: "daily" }
  open-pull-requests-limit: 10
  ignore:
    - dependency-name: "org.jetbrains.kotlin*"
      update-types: ["version-update:semver-major"]
    - dependency-name: "io.ktor:*"
      update-types: ["version-update:semver-major"]
    - dependency-name: "com.android.application"
      update-types: ["version-update:semver-major"]
```

## Evaluation result

| Metric | Value |
|---|---|
| Kotlin files scanned | 32 |
| Pipeline `OK` | 10 |
| Pipeline `BLOCKED_BUILD` | 1 (`gradle-wrapper 8 → 9.5.1`) |
| Static-analysis F1 | 0.780 (Recall = **1.00**) |
| DroidBot UTGs generated | 10/10 |

Recall is perfect here: the analyzer does not miss any file that the ground truth marks as impacted. Precision is moderate because some bumps touch project-wide configuration (AGP, `com.github.ben-manes.versions`) and the static phase reports more files than the ground truth — a conservative-by-design behaviour rather than a defect.
