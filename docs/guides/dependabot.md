# Configuring Dependabot

KMP-IMPACT depends on Dependabot to surface version bumps as pull requests against `gradle/libs.versions.toml`. The recommended configuration biases Dependabot toward minor and patch bumps so the AFTER APK keeps compiling — which keeps the dynamic phase usable.

## Why bias the configuration?

Major bumps of foundational tooling (Kotlin, Compose, Coroutines, Serialization, Ktor, AGP, KSP) routinely introduce ABI breaks. When that happens:

1. The AFTER APK fails to build.
2. Phase 3 emits `BLOCKED — APK assembly failed (AFTER)`.
3. The dynamic tab in the report shows the Gradle log instead of a UTG diff.

Patch and minor bumps are far more likely to keep the build healthy, which preserves end-to-end coverage of the pipeline.

## Reference configuration

```yaml title=".github/dependabot.yml"
--8<-- "examples/github-action/dependabot.yml"
```

## Field-by-field explanation

| Field | What it controls |
|---|---|
| `package-ecosystem: "gradle"` | Tells Dependabot to read Gradle dependencies — including the version catalog. |
| `directory: "/"` | Project root. Set to a subdirectory for monorepos. |
| `schedule.interval: "daily"` | How often Dependabot scans for updates. |
| `open-pull-requests-limit: 10` | Cap on concurrent open PRs. Keeps Pages deploy concurrency manageable. |
| `ignore[].dependency-name` | Pattern to match Maven coordinates. Wildcards allowed. |
| `ignore[].update-types` | `version-update:semver-major`, `version-update:semver-minor`, or `version-update:semver-patch`. |

## Recommended `ignore` rules

```yaml
ignore:
  - dependency-name: "org.jetbrains.kotlin*"
    update-types: ["version-update:semver-major"]
  - dependency-name: "org.jetbrains.kotlinx:kotlinx-coroutines*"
    update-types: ["version-update:semver-major"]
  - dependency-name: "org.jetbrains.kotlinx:kotlinx-serialization*"
    update-types: ["version-update:semver-major"]
  - dependency-name: "org.jetbrains.compose*"
    update-types: ["version-update:semver-major"]
  - dependency-name: "androidx.compose*"
    update-types: ["version-update:semver-major", "version-update:semver-minor"]
  - dependency-name: "androidx.compose:compose-bom"
    update-types: ["version-update:semver-major"]
  - dependency-name: "io.ktor:*"
    update-types: ["version-update:semver-major"]
  - dependency-name: "com.android.application"
    update-types: ["version-update:semver-major"]
  - dependency-name: "com.android.library"
    update-types: ["version-update:semver-major"]
  - dependency-name: "com.google.devtools.ksp"
    update-types: ["version-update:semver-major"]
```

## Also ignore the Gradle wrapper major bump

Bumps from Gradle 8.x to 9.x routinely break AGP 8.x. Adding the wrapper to the ignore list prevents the corresponding PR from showing up as `BLOCKED_BUILD`:

```yaml
- dependency-name: "gradle-wrapper"
  update-types: ["version-update:semver-major"]
```

## Multiple ecosystems

A KMP project may have ancillary tooling under other ecosystems — for example, a Python helper for SBOM generation. Declare each ecosystem as its own block:

```yaml
- package-ecosystem: "pip"
  directory: "/pipeline/sbom"
  schedule:
    interval: "weekly"
  open-pull-requests-limit: 2
```

PRs opened for non-Gradle ecosystems are caught by the `detect` job in the workflow and reported as `EXPECTED_SKIPPED` rather than as failures.

## See also

- [Reference → GitHub Action](../reference/github-action.md) — how the `detect` job decides whether to continue.
- [Troubleshooting → L1](../troubleshooting.md#l1-direct-buildgradlekts-versions-are-not-detected) — what happens when versions live outside the catalog.
