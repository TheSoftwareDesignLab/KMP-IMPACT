# Analyzing a bump locally

Run the pipeline against any Kotlin Multiplatform repository on your machine. Useful for triaging a real Dependabot PR before the CI workflow finishes, or for experimenting with a bump that has not been raised yet.

## Prerequisites

- KMP-IMPACT installed in editable mode (see [Installation](../getting-started/installation.md)).
- A KMP project that keeps its versions in `gradle/libs.versions.toml`.
- For the dynamic phase: JDK 21, the Android SDK, and a running emulator.

## Static-only analysis

The fastest mode. Skips Phase 3 entirely.

```bash
kmp-impact analyze \
  --repo /path/to/your/kmp/project \
  --dependency io.ktor \
  --before-version 2.3.8 \
  --after-version 2.3.11 \
  --output-dir output \
  --skip-dynamic
```

`--dependency` accepts a Maven group prefix (`io.ktor`) or a full coordinate substring (`io.ktor:ktor-client-core`). The analyzer resolves it to one or more Kotlin package roots through the internal mapping table.

## Full analysis with the dynamic phase

Drop `--skip-dynamic`. The pipeline will:

1. Build `./gradlew :<android-module>:assembleDebug` on the BEFORE state.
2. Build the same target on the AFTER state.
3. Launch DroidBot against each APK and diff the resulting UTGs.

```bash
kmp-impact analyze \
  --repo /path/to/your/kmp/project \
  --dependency io.ktor \
  --before-version 2.3.8 \
  --after-version 2.3.11 \
  --output-dir output-full
```

If either APK fails to compile, the pipeline emits a `BLOCKED` Phase 3 with the Gradle log attached. The static phase still completes.

## Opening the report

```bash
open output/report/index.html         # macOS
xdg-open output/report/index.html     # Linux
start  output/report/index.html       # Windows
```

The six tabs are documented in [Guides → Reading the Report](reading-the-report.md).

## Re-running a single phase

Each phase reads the previous phase's JSON from disk, so you can re-run a single phase by pointing the CLI at an existing `output/` directory. This is useful when iterating on `phase4_consolidate` or `phase5_visualize` without re-running the expensive static scan.

```bash
kmp-impact analyze \
  --repo /path/to/project \
  --dependency io.ktor \
  --before-version 2.3.8 \
  --after-version 2.3.11 \
  --output-dir output \
  --skip-dynamic \
  --keep-shadows
```

`--keep-shadows` retains `phase1/before/` and `phase1/after/`, which lets you re-execute Phase 2 against the same shadow copies.

## See also

- [API Reference → CLI](../reference/cli/index.md) — every flag in detail.
- [Architecture → Pipeline Overview](../architecture/pipeline-overview.md) — how the phases compose.
- [Troubleshooting](../troubleshooting.md) — diagnosis flows for `BLOCKED` phases.
