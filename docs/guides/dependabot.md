# Configuring Dependabot

KMP-IMPACT relies on Dependabot to surface version bumps as pull requests against `gradle/libs.versions.toml`. If you have never set up Dependabot before, the primer below gets you to a working configuration in under five minutes; the rest of the page covers the biasing that keeps the analyzer's AFTER APK building.

## A 30-second primer

**What it is.** Dependabot is a GitHub-managed service that scans your repository for outdated dependencies and opens pull requests to update them. It is free for public and private repositories and ships out of the box — there is nothing to install or download.

**What it scans.** Dependabot supports many ecosystems. For KMP-IMPACT, only two matter:

- `gradle` — reads Gradle build files (including the version catalog at `gradle/libs.versions.toml`) and bumps Maven coordinates.
- `github-actions` — reads workflows under `.github/workflows/` and bumps third-party Action versions.

A third one, `pip`, is sometimes useful if your repository hosts Python helpers (the analyzer itself is published this way).

**How you enable it.** Commit a file at `.github/dependabot.yml`. That's the entire activation. The next scan happens within an hour.

## Quick start (5-minute setup)

1. **Create the configuration.** Copy the analyzer's reference file:

    ```bash
    cp tools/kmp-impact-analyzer/examples/github-action/dependabot.yml .github/dependabot.yml
    ```

2. **Verify Dependabot has the necessary permission.** In *Settings → Code security → Dependabot*, make sure *Dependabot alerts* and *Dependabot version updates* are enabled.

3. **Commit and push.** The next scan runs within an hour and opens the first batch of PRs.

4. **Inspect the open PRs.** Each PR opened by Dependabot for a Gradle bump will trigger the `impact-analysis.yml` workflow automatically. See [How everything talks to each other](how-it-works.md) for the full flow.

That's it. There is no Dependabot CLI to install and no service to run — GitHub does the scheduling and the PR creation.

## How KMP-IMPACT uses Dependabot's PRs

When Dependabot opens a PR, the diff modifies `gradle/libs.versions.toml`. The reference workflow's `paths:` filter matches that path, so GitHub Actions schedules a run. The pipeline:

1. Detects the changed `dependency_group`, `before_version`, `after_version`.
2. Materialises BEFORE and AFTER shadow copies.
3. Runs static and dynamic analysis in parallel.
4. Publishes the HTML report and comments on the PR.

The reviewer reads the comment, opens the linked report, and decides. Dependabot's role ends as soon as the PR is open — it does not interact with the analyzer directly.

The interaction diagram and a step-by-step walkthrough live in [How everything talks to each other](how-it-works.md).

## Why bias the configuration?

Major bumps of foundational tooling (Kotlin, Compose, Coroutines, Serialization, Ktor, AGP, KSP) routinely introduce ABI breaks. When that happens:

1. The AFTER APK fails to build.
2. Phase 3 emits `BLOCKED — APK assembly failed (AFTER)`.
3. The dynamic tab in the report shows the Gradle log instead of a UTG diff.

Patch and minor bumps are far more likely to keep the build healthy, which preserves end-to-end coverage of the pipeline. The recommended `.github/dependabot.yml` therefore *narrows* what Dependabot opens — it does not expand it.

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

## Troubleshooting Dependabot itself

| Symptom | Likely cause |
|---|---|
| No PRs open after 24 hours. | `.github/dependabot.yml` not present on the default branch, or *Dependabot version updates* disabled in Settings. |
| Dependabot bumps something you wanted ignored. | The `ignore` glob does not match the coordinate. Check it against the Maven coordinate the catalog uses. |
| Dependabot keeps reopening a closed PR. | Closing a Dependabot PR is the way to signal "don't update this dependency yet"; if you want it gone permanently, add it to `ignore`. |
| The PR is opened but the workflow doesn't run. | The bump didn't reach `gradle/libs.versions.toml`. See [L1](../troubleshooting.md#l1-direct-buildgradlekts-versions-are-not-detected). |

## See also

- [How everything talks to each other](how-it-works.md) — the end-to-end Dependabot ↔ workflow ↔ analyzer flow.
- [Configuring GitHub Actions](github-actions.md) — installation walkthrough.
- [Reference → GitHub Action](../reference/github-action.md) — workflow inputs, outputs, and permissions.
