# Pokedex

A Compose Multiplatform Pokédex application targeting Android and Desktop. Used as the reference preset because its layout is the canonical KMP application structure.

- Reference repo: [`EstebanCastel/KMP-IMPACT-Pokedex-with-tool`](https://github.com/EstebanCastel/KMP-IMPACT-Pokedex-with-tool)
- Baseline (no tool): [`EstebanCastel/KMP-IMPACT-Pokedex-baseline`](https://github.com/EstebanCastel/KMP-IMPACT-Pokedex-baseline)

## Stack

| Component | Version |
|---|---|
| AGP | 8.5 |
| Gradle wrapper | 8.10.2 |
| Kotlin | 2.0.21 |
| Compose Compiler plugin | applied (mandatory under Kotlin 2.x) |
| Targets | `android`, `desktop`, `shared` (commonMain, androidMain, desktopMain, iosMain) |

## Catalog highlights

```toml title="gradle/libs.versions.toml"
[versions]
kotlin = "2.0.21"
agp    = "8.5.0"

[plugins]
kotlin-multiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
kotlin-compose       = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
android-application  = { id = "com.android.application",            version.ref = "agp" }
```

Kotlin 2.x requires the Compose Compiler plugin to be applied explicitly in every module that uses Compose:

```kotlin title="android/build.gradle.kts"
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.multiplatform)
    alias(libs.plugins.kotlin.compose)
}

kotlin {
    jvmToolchain(11)
    androidTarget { … }
}
```

`jvmToolchain(11)` lives at the top of the `kotlin { … }` block — **not** inside a target. The same applies to `shared/build.gradle.kts` and `desktop/build.gradle.kts`.

## Android module discovery

The pipeline detects the Android module by trying known names in order: `:android`, `:app`, `:composeApp`, `:androidApp`, `:android-app`, `:shared:android`, `:app:android`. Pokedex matches `:android`.

If your module is named differently, set:

```yaml title=".github/workflows/impact-analysis.yml"
env:
  KMP_IMPACT_ANDROID_MODULE: ":your-android-module"
```

## Dependabot biasing

```yaml title=".github/dependabot.yml"
version: 2
updates:
  - package-ecosystem: "gradle"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 10
    ignore:
      - dependency-name: "org.jetbrains.kotlin*"
        update-types: ["version-update:semver-major"]
      - dependency-name: "androidx.compose:compose-bom"
        update-types: ["version-update:semver-major"]
      - dependency-name: "com.android.application"
        update-types: ["version-update:semver-major"]
      - dependency-name: "io.ktor:*"
        update-types: ["version-update:semver-major"]
```

Major bumps of Kotlin, Compose BOM, AGP and Ktor are blocked because they routinely introduce ABI breaks that prevent the AFTER APK from building. Patch and minor bumps still flow.

## Evaluation result

| Metric | Value |
|---|---|
| PRs opened by Dependabot | 10 |
| Pipeline `OK` | 8 |
| Pipeline `BLOCKED_BUILD` | 1 (`gradle-wrapper 8.x → 9.5.1`) |
| Pipeline `EXPECTED_SKIPPED` | 1 (`pip/requests`) |
| Static-analysis F1 | 0.699 |
| DroidBot UTGs generated | 8/8 |

Notable finding: the analyzer does not currently map `io.github.qdsfdhvh` to `com.seiko.imageloader`, missing one direct file in the `image-loader` bump. Adding the mapping moves Pokedex's F1 up.
