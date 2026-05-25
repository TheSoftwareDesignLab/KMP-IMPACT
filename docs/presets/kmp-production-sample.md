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

## Why this preset is a good fit

Useful as a control case for projects that do **not** use Compose. ABI breaks tied to Compose Compiler or BOM bumps do not apply, so the workflow gets fewer `BLOCKED_BUILD` outcomes. Recall on Ktor and Kotlinx dependencies tends to be excellent: the BFS over imports captures every file that touches the bumped library.
