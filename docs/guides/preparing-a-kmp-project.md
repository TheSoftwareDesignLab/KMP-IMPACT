# Preparing an existing KMP project

The [Configuring GitHub Actions](github-actions.md) guide has only three steps because it assumes your KMP project already meets the prerequisites in [Requirements](../getting-started/requirements.md). Most production repos do not — Kotlin 2.x rolled out new conventions, Compose Multiplatform tightened its plugin requirements, and many forks inherit upstream workflows that depend on secrets you don't own.

This page is the **before-the-install checklist**: nine concrete tasks to convert an arbitrary KMP repository into one where the pipeline can run cleanly. Skip the items that already apply to your project.

<div class="kmp-callout" markdown>
**Quick self-assessment.** If you can answer "yes" to all of these, jump straight to [Configuring GitHub Actions](github-actions.md):

1. Your project uses `gradle/libs.versions.toml` as the single source of truth.
2. The Gradle wrapper version is **8.7+** for AGP 8.x or **9.0+** for AGP 9.x.
3. The CI workflow runs on **JDK 21**.
4. If you use Kotlin 2.x with Compose, the `org.jetbrains.kotlin.plugin.compose` plugin is applied to every Compose module.
5. `jvmToolchain(…)` is at the top of the `kotlin { … }` block, not inside a target.
6. You have an Android application module named `shared`, `composeApp`, `androidApp`, `app`, `common`, `kmm-shared`, or `kmpShared`.
7. `./gradlew :<android-module>:assembleDebug` succeeds locally.
8. The repository's *Settings → Actions* permissions include `contents: write`, `pull-requests: write`, `pages: write`, `id-token: write`.
9. Your fork doesn't have upstream workflows that fail for lack of secrets.

Each section below covers one item.
</div>

## 1. Use a Gradle version catalog

KMP-IMPACT detects bumps from diffs against `gradle/libs.versions.toml`. If your project pins versions directly in `build.gradle.kts`, `gradle.properties`, `pluginManagement { … }`, or `buildSrc/`, the analyzer's `detect-version-changes` step will not see them — see [L1](../troubleshooting.md#l1-direct-buildgradlekts-versions-are-not-detected).

**Action.** Migrate your dependencies to a version catalog. Minimal example:

```toml title="gradle/libs.versions.toml"
[versions]
kotlin = "2.0.21"
agp    = "8.5.0"
ktor   = "2.3.11"

[libraries]
ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktor" }

[plugins]
kotlin-multiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
android-application  = { id = "com.android.application",            version.ref = "agp" }
```

Then reference aliases from every `build.gradle.kts`:

```kotlin
plugins {
    alias(libs.plugins.kotlin.multiplatform)
}

dependencies {
    implementation(libs.ktor.client.core)
}
```

**Verify.** Open a draft PR that bumps any version in `libs.versions.toml`. The `paths:` filter on the reference workflow must match, so the run should queue.

## 2. Align JDK, Gradle, AGP, and Kotlin

The four versions interact. Use the matrix in [Requirements → Compatibility](../getting-started/requirements.md#compatibility-matrix) to pick a stack that has been validated end-to-end. The most common breakage is a Gradle wrapper that is too old for the AGP it ships with.

**Action.**

- Set the CI JDK to 21 in the workflow (the reference workflow already does this).
- Update `gradle/wrapper/gradle-wrapper.properties` to a Gradle version compatible with your AGP:

  ```properties title="gradle/wrapper/gradle-wrapper.properties"
  distributionUrl=https\://services.gradle.org/distributions/gradle-8.10.2-bin.zip
  ```

- If your AGP version is 9.x, the wrapper must be 9.0+.

**Verify.**

```bash
./gradlew --version
# Expect: Gradle 8.10.2 (or 9.x for AGP 9)
#         JVM: 21.x (when running through the CI workflow)
```

## 3. Apply the Compose Compiler plugin under Kotlin 2.x

Kotlin 2.0 changed how Compose support is wired. Without an explicit `org.jetbrains.kotlin.plugin.compose` declaration, the Android module fails to compile — and Phase 3 marks the dynamic phase `BLOCKED` because no APK is produced.

**Action.** Register the plugin in the catalog:

```toml title="gradle/libs.versions.toml"
[plugins]
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
```

Apply it in the root `build.gradle.kts` (so Gradle resolves it):

```kotlin title="build.gradle.kts (root)"
plugins {
    alias(libs.plugins.kotlin.compose) apply false
}
```

And in every module that uses Compose:

```kotlin title="<compose-module>/build.gradle.kts"
plugins {
    alias(libs.plugins.kotlin.compose)
}
```

This typically means at least `android/build.gradle.kts`, plus `shared/build.gradle.kts` and any platform-specific Compose modules (`desktop/`, `composeApp/`, …).

**Skip this section if** you are on Kotlin 1.9.x — the Compose Multiplatform plugin still handles Compose Compiler internally.

**Verify.**

```bash
./gradlew :<android-module>:tasks --all | grep -i compose
# Should list Compose-related tasks such as :compileDebugKotlinAndroid with Compose plugin metadata
```

## 4. Move `jvmToolchain` to the top of the `kotlin { … }` block

Kotlin 2.x rejects `jvmToolchain(…)` calls inside a specific target. The placement must be at the extension scope:

```kotlin
kotlin {
    jvmToolchain(11)         //  <-- here

    jvm { withJava() }
    androidTarget { ... }
}
```

**Not** inside a target:

```kotlin
kotlin {
    jvm {
        jvmToolchain(11)     //  <-- breaks under Kotlin 2.x
    }
}
```

**Action.** Audit every `kotlin { … }` block in your project (root, `shared/`, `android/`, `desktop/`, …) and hoist the `jvmToolchain(…)` call to the top.

**Verify.**

```bash
./gradlew :shared:compileKotlinMetadata --info
# Look for the "jvmToolchain" line; no warnings about deprecated placement
```

## 5. Make sure your Android module is discoverable

The reference workflow probes the following module names, in this order:

```text
shared, composeApp, androidApp, app, common, kmm-shared, kmpShared
```

If none of these match, it falls back to `:app`. A module named something else — `:mobile`, `:client-android`, etc. — will not be detected, and the `droidbot` job marks itself `BLOCKED — Android module not detected`.

**Action.** Pick the option that requires the least churn:

- **Easiest.** Rename your existing Android module to one of the names above. Most projects can move to `:app` or `:composeApp` with a one-line `settings.gradle.kts` change.
- **Alternative.** Add an alias module that re-exposes your real Android module:

  ```kotlin title="settings.gradle.kts"
  include(":app")
  project(":app").projectDir = file("client/android")   // your real path
  ```

- **Last resort.** Edit the `Detect Android app module` step in `.github/workflows/impact-analysis.yml` and add your module name to the loop. You lose the upstream upgrade path, so prefer one of the previous two options.

**Verify.**

```bash
./gradlew projects | grep -E ':(shared|composeApp|androidApp|app|common|kmm-shared|kmpShared)\b'
# Should list at least one of the seven canonical names
```

## 6. Make sure the Android module is an *application*

The reference workflow runs `./gradlew :<module>:assembleDebug` and expects a real APK to land under `*/build/outputs/apk/debug/*.apk`. A module that applies only `com.android.library` will not produce one.

**Action.** Ensure the detected module applies `com.android.application` and declares an `applicationId`:

```kotlin title="<android-module>/build.gradle.kts"
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
}

android {
    namespace = "com.example.app"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.app"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }
}
```

If the only Android-related module in your project is a library, you will not be able to run the dynamic phase. The static phase still works — use `--skip-dynamic` locally and accept the `BLOCKED — APK assembly failed` status in CI.

**Verify.**

```bash
./gradlew :<android-module>:assembleDebug
ls <android-module>/build/outputs/apk/debug/*.apk
# Should print at least one .apk file
```

## 7. Disable upstream workflows that need secrets you don't own

This step only matters if you forked your KMP project from an upstream repository. Many open-source KMP apps (Confetti, DroidconKotlin, …) ship workflows that publish to Google Cloud, App Store Connect, Firebase, or sign Android releases. Without those secrets, every workflow run fails *before* doing useful work — your PRs end up with rows of red checks even when `impact-analysis.yml` runs cleanly.

**Action.** Disable every workflow except `impact-analysis.yml` in one shot:

```bash
gh api repos/<owner>/<repo>/actions/workflows --jq \
  '.workflows[]
    | select(.path | startswith(".github/workflows/"))
    | select(.path != ".github/workflows/impact-analysis.yml")
    | .id' \
| while read wid; do
    gh api -X PUT "repos/<owner>/<repo>/actions/workflows/$wid/disable"
  done
```

This keeps the workflow files on disk but prevents them from triggering. You can re-enable individual ones later if you provide the missing secrets.

**Verify.** Open the *Actions* tab on a freshly opened Dependabot PR — only the *Dependency Impact Analysis* workflow should run.

## 8. Set the right Actions permissions on the repository

In *Settings → Actions → General*:

| Setting | Value |
|---|---|
| Workflow permissions | Read and write permissions |
| Allow GitHub Actions to create and approve pull requests | enabled (Dependabot relies on this) |

In *Settings → Pages*:

| Setting | Value |
|---|---|
| Source | GitHub Actions |

Or, equivalently, via the CLI once:

```bash
gh api -X PUT "repos/<owner>/<repo>/pages" -f "build_type=workflow"
```

**Verify.**

```bash
gh api "repos/<owner>/<repo>/pages" --jq '.build_type'
# Expect: "workflow"
```

## 9. (Optional) Add a `pipeline/` directory if you want SBOM scanning

The reference workflow's `paths:` filter includes `pipeline/**`, which is used by some projects to host SBOM generation scripts. The Pokedex preset, for example, has a `pipeline/sbom/requirements.txt` that Dependabot tracks under the `pip` ecosystem.

**This is purely optional.** If your project does not need an SBOM tracker, leave the directory absent — the workflow simply ignores the path filter entry.

If you do want it, mirror the reference layout:

```text
pipeline/
└── sbom/
    └── requirements.txt
```

And declare the matching Dependabot block:

```yaml title=".github/dependabot.yml"
- package-ecosystem: "pip"
  directory: "/pipeline/sbom"
  schedule:
    interval: "weekly"
```

PRs opened for non-Gradle ecosystems are caught by the `detect` job and reported as `EXPECTED_SKIPPED` — they are not failures.

## Final verification: a 30-second smoke test

When all nine items pass, run this from the project root before installing KMP-IMPACT:

```bash
# 1. Catalog exists
test -f gradle/libs.versions.toml || echo "FAIL: no version catalog"

# 2. Gradle + JDK
./gradlew --version | grep -E '^(Gradle|JVM)'

# 3. Android module discoverable + APK builds
MODULE=$(./gradlew projects 2>/dev/null \
  | grep -oE "':(shared|composeApp|androidApp|app|common|kmm-shared|kmpShared)'" \
  | head -1 | tr -d "':")
echo "Detected Android module: :${MODULE:-app}"
./gradlew ":${MODULE:-app}:assembleDebug" || echo "FAIL: assembleDebug"
```

If all three checks pass, you are ready to follow [Configuring GitHub Actions](github-actions.md).

## See also

- [Getting Started → Requirements](../getting-started/requirements.md) — the full compatibility matrix.
- [Configuring GitHub Actions](github-actions.md) — the three-step install once your project is prepared.
- [How everything talks to each other](how-it-works.md) — the runtime flow your project will participate in.
- [Troubleshooting](../troubleshooting.md) — what `BLOCKED` phases mean if something slips through.
