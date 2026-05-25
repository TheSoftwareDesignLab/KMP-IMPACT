# Requirements

KMP-IMPACT has two execution modes — *static-only* and *full* (with the dynamic DroidBot phase). The dynamic phase has stricter prerequisites; the static phase runs anywhere Python 3.10+ does.

## Host machine

| Component | Minimum | Notes |
|---|---|---|
| Python | **3.10** | The package targets `>=3.10`. CI tests `3.10`, `3.11`, `3.12`. |
| JDK | **21** | Required by AGP 8.x. Use Temurin/Adoptium; do not use Oracle JDK 22+. |
| Gradle | **8.7+** for AGP 8.x — **9.0+** for AGP 9.x | The static phase reuses the project's own Gradle wrapper. |
| Disk | ~2 GB per analysis | Two shadow copies of the project + build outputs. |
| Network | Outbound HTTPS | Maven Central, Google Maven, Gradle plugin portal. |

## Target KMP repository

The analyzer inspects a real KMP project. For the dynamic phase to run, the project must:

- Declare its versions in `gradle/libs.versions.toml` (single source of truth).
- Apply the Compose Compiler plugin explicitly when using **Kotlin 2.x** (otherwise the Android APK won't build):

  ```kotlin title="build.gradle.kts (root)"
  alias(libs.plugins.kotlin.compose) apply false
  ```

  ```toml title="gradle/libs.versions.toml"
  kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
  ```

- Place `jvmToolchain(11)` inside the `kotlin { … }` block (top-level, not inside `compilerOptions`) when on Kotlin 2.x.
- Expose an Android module that produces a Debug APK (`./gradlew :android:assembleDebug`).

A full reference of these prerequisites lives in the project repository at [`CONDICIONES_MINIMAS_DROIDBOT_KMP.md`](https://github.com/EstebanCastel/KMP-IMPACT-Pokedex-with-tool/blob/main/CONDICIONES_MINIMAS_DROIDBOT_KMP.md).

## Dynamic phase (DroidBot)

| Component | Minimum | Notes |
|---|---|---|
| Android SDK | API 33+ | `cmdline-tools`, `platform-tools`, `emulator`. |
| AVD | x86_64, API 33+ | The CI workflow boots `system-images;android-33;default;x86_64`. |
| adb | Recent | The runner uses adb shell instrumentation. |

In CI (GitHub Actions) the [reference workflow](../guides/github-actions.md) handles all of this for you. Locally, you only need the dynamic phase if you are debugging UI regressions interactively — most analyses run fine with `--skip-dynamic`.

## What if my project doesn't meet the prerequisites?

KMP-IMPACT will tell you. Whenever the APK cannot be built or DroidBot cannot reach a UTG, the pipeline emits a `BLOCKED` status with an explicit `blocked_reason` rather than producing a mock result.

See [Troubleshooting](../troubleshooting.md) for the catalogue of known blockers.
