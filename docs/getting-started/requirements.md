# Requirements

KMP-IMPACT has two execution modes — *static-only* and *full* (with the dynamic DroidBot phase). The dynamic phase has stricter prerequisites; the static phase runs anywhere Python 3.10+ does.

## Host machine

| Component | Minimum | Notes |
|---|---|---|
| Python | **3.10** | The package targets `>=3.10`. CI tests `3.10`, `3.11`, `3.12`. |
| JDK | **21** | Required by AGP 8.x. Use Temurin / Adoptium; do not use Oracle JDK 22+. |
| Gradle | **8.7+** for AGP 8.x — **9.0+** for AGP 9.x | The static phase reuses the project's own Gradle wrapper. |
| Disk | ~2 GB per analysis | Two shadow copies of the project + build outputs. |
| Network | Outbound HTTPS | Maven Central, Google Maven, Gradle plugin portal. |

The exact list of Python packages pulled by `pip install -e ".[dev]"` is documented in [What gets installed](what-gets-installed.md).

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

## Compatibility matrix

The combinations below have all been exercised end-to-end on the reference [presets](../presets/index.md). Other combinations may work but are not validated.

### Kotlin

| Kotlin version | Static phase | Dynamic phase | Notes |
|---|---|---|---|
| 1.9.21 – 1.9.24 | yes | yes | No `kotlin.plugin.compose` requirement; Compose handled by the Compose Multiplatform plugin. |
| 2.0.x | yes | yes | **Compose Compiler plugin required** for any module that uses Compose. `jvmToolchain` must live at the top of the `kotlin { … }` block. |
| 2.1.x | yes | yes (untested in CI) | Same caveats as 2.0.x. |
| 2.2.x | yes | yes | Verified on Confetti and DroidconKotlin presets. |
| 2.3.x | yes | yes (untested) | Newer than the validated set. The static phase should be unaffected; the dynamic phase depends on AGP + Gradle support for the Kotlin version. |

### Android Gradle Plugin (AGP)

| AGP version | Minimum Gradle | Notes |
|---|---|---|
| 8.1.x | 8.5 | Verified on kmp-production-sample. |
| 8.5.x | 8.7 (8.10.2 recommended) | Verified on Pokedex and KMP-App-Template. |
| 8.13.x | 9.3.1 | Verified on Confetti. |
| 8.14.x | 8.14.3 | Verified on DroidconKotlin. |
| 9.x | 9.0+ | Supported; major bumps from AGP 8 to 9 typically require a matching Gradle 9 wrapper. |

The Gradle wrapper version is enforced at the start of Phase 1. Mismatches surface as a `WARNING` in `phase1/manifest.json` and, in CI, as a workflow step that upgrades the wrapper in place for the duration of the run.

### Gradle

| Gradle version | Compatible with | Notes |
|---|---|---|
| 8.5 | AGP ≤ 8.1.x | Older line. Works for kmp-production-sample. |
| 8.7 – 8.10.2 | AGP 8.x | Common choice. 8.10.2 is the version validated on Pokedex. |
| 8.14.x | AGP 8.13–8.14 | Validated on DroidconKotlin. |
| 9.x | AGP 8.13+ or AGP 9.x | Required line for AGP 9 and recommended for newer AGP 8 releases. |

### KMP target platforms

| Target | Static phase | Dynamic phase |
|---|---|---|
| `androidMain` | yes — source-set aware | yes — DroidBot UTG diff |
| `commonMain` | yes | n/a (no per-target runtime) |
| `iosMain` | yes — source-set aware | not supported |
| `desktopMain` | yes — source-set aware | not supported |
| `jvmMain` | yes — source-set aware | not supported |
| `jsMain` / `wasmJsMain` | yes — source-set aware | not supported |
| `commonTest`, `androidTest`, `iosTest` | yes — source-set aware | not supported |

The static phase walks every `.kt` file under any source set. The dynamic phase only knows how to drive Android emulators; iOS, Desktop, JVM, and JS clients are out of scope. UI regressions on those targets are not detected by KMP-IMPACT.

## Dynamic phase (DroidBot)

| Component | Minimum | Notes |
|---|---|---|
| Android SDK | API 33+ | `cmdline-tools`, `platform-tools`, `emulator`. |
| AVD | x86_64, API 33+ | The CI workflow boots `system-images;android-33;default;x86_64`. |
| adb | Recent | The runner uses adb shell instrumentation. |

In CI (GitHub Actions) the [reference workflow](../guides/github-actions.md) handles all of this for you. Locally, you only need the dynamic phase if you are debugging UI regressions interactively — most analyses run fine with `--skip-dynamic`.

## What if my project doesn't meet the prerequisites?

Two options:

- **Adapt your project.** Follow [Guides → Preparing an existing KMP project](../guides/preparing-a-kmp-project.md) — a nine-step checklist that takes an arbitrary KMP repo from "as-is" to "ready to install KMP-IMPACT".
- **Ship as-is and accept the limitations.** KMP-IMPACT will tell you when something is wrong. Whenever the APK cannot be built or DroidBot cannot reach a UTG, the pipeline emits a `BLOCKED` status with an explicit `blocked_reason` rather than producing a mock result.

See [Troubleshooting](../troubleshooting.md) for the catalogue of known blockers and the matching diagnosis flow.
