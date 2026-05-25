# Diagnosing a BLOCKED phase

When a phase cannot produce real data, the pipeline emits `BLOCKED` with an explicit `blocked_reason` rather than fabricating output. This guide walks through the most common cases and the order to check them.

<div class="kmp-callout" markdown>
A `BLOCKED` phase is **not** an error. It means the analyzer correctly detected that it could not produce real data for that phase. The report still publishes; the affected tab carries the failure reason.
</div>

## Quick triage table

| Symptom | Most likely cause | Detailed guide |
|---|---|---|
| `Detect version change` says `no Gradle catalog change found` | The PR doesn't touch `gradle/libs.versions.toml`. | [L1](../troubleshooting.md#l1-direct-buildgradlekts-versions-are-not-detected) |
| Report shows `0 impacted files` for an AGP / KSP / wrapper bump | Build plugin — no Kotlin imports to walk. | [L2](../troubleshooting.md#l2-plugin_or_toolchain-bumps-report-zero-static-impact) |
| `source_set = "unknown"` in the report | Non-convention source layout. | [L3](../troubleshooting.md#l3-non-convention-source-set-layout) |
| `Dynamic analysis: BLOCKED — DroidBot produced no UTG artifact` | No APK or DroidBot failure. | [L4](../troubleshooting.md#l4-droidbot-blocked-no-apk) |
| `Deploy report to GitHub Pages` ends `cancelled` | Concurrent Pages deploys. | [L6](../troubleshooting.md#l6-deploy-pages-cancellation) |
| Static F1 unexpectedly low for Compose Multiplatform | Mapping gap. | [L7](../troubleshooting.md#l7-compose-multiplatform-mapping) |
| Files that use DI bindings are not flagged | DI not in the import graph. | [L8](../troubleshooting.md#l8-dependency-injection-misses) |

## DroidBot `BLOCKED` — triage order

This is the most common case. Check in order:

1. **`Detect Android app module`** — must resolve a real module. The workflow probes `shared, composeApp, androidApp, app, common, kmm-shared, kmpShared` in that order, then falls back to `:app`. If none of these match your real module name, expose it under one of those aliases.
2. **`Ensure Gradle wrapper is recent enough for AGP`** — must show ≥ 8.7 for AGP 8.x and ≥ 9.0 for AGP 9.x.
3. **`Build BEFORE APK`** — must end with `BEFORE APK: /tmp/before.apk`.
4. **`Build AFTER APK`** — same expectation.
5. **UTG artifact** — `phase3/before-utg/` and `phase3/after-utg/` must contain `utg.js` or `utg.json`. Empty directories are rejected on purpose.

## Kotlin 2.x without the Compose Compiler plugin

If the project uses Kotlin 2.x with Compose and the AFTER APK fails to build, the most common cause is a missing `org.jetbrains.kotlin.plugin.compose` plugin. Required additions:

```toml title="gradle/libs.versions.toml"
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
```

```kotlin title="<module>/build.gradle.kts"
plugins {
    alias(libs.plugins.kotlin.compose)
}
```

Also place `jvmToolchain(…)` at the top of the `kotlin { … }` block, not inside a target.

## Pages cancellation — recovery

`actions/deploy-pages@v4` enforces concurrency at the repository level. When several Dependabot PRs finish at the same time, newer deploys cancel older ones.

Recovery:

```bash
gh run rerun <run-id> -R <owner>/<repo>
gh run watch    <run-id> -R <owner>/<repo>
```

For multi-PR bursts, re-run cancelled deploys serially — wait for each `gh run watch` to return before launching the next one.

## See also

- [Troubleshooting](../troubleshooting.md) — the canonical list of every documented limitation.
- [Reference → `UIRegressions`](../reference/contracts/ui-regressions.md) — the `blocked` status enum.
