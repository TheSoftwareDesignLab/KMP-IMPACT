# `kmp-impact analyze`

Run the full pipeline against a single dependency bump.

## Synopsis

```bash
kmp-impact analyze \
  --repo PATH \
  --dependency NAME \
  --before-version A \
  --after-version B \
  [--output-dir OUT] \
  [--skip-dynamic] \
  [--before-apk PATH] \
  [--after-apk PATH] \
  [--droidbot-before-output PATH] \
  [--droidbot-after-output PATH]
```

## Flags

| Flag | Required | Default | Description |
|---|---|---|---|
| `--repo` | yes | — | Path or URL to the KMP project repository. |
| `--dependency` | yes | — | Dependency group (e.g. `io.ktor`, `org.jetbrains.compose`). |
| `--before-version` | yes | — | Version before the change. |
| `--after-version` | yes | — | Version after the change. |
| `--output-dir` | no | `output` | Directory where shadow copies, phase outputs, and the HTML report are written. |
| `--skip-dynamic` | no | off | Skip Phase 3 (DroidBot). The report still publishes; the dynamic tab shows `skipped`. |
| `--before-apk` | no | — | Path to a pre-built APK for the BEFORE state. Skips the Gradle build for that side. |
| `--after-apk` | no | — | Path to a pre-built APK for the AFTER state. Skips the Gradle build for that side. |
| `--droidbot-before-output` | no | — | Path to a pre-generated DroidBot output directory for the BEFORE APK. Skips the DroidBot run for that side. |
| `--droidbot-after-output` | no | — | Path to a pre-generated DroidBot output directory for the AFTER APK. Skips the DroidBot run for that side. |

The four optional dynamic-phase shortcuts (`--*-apk`, `--droidbot-*-output`) are useful for two cases:

- **CI reuse.** When the CI matrix has already built the APK in a previous job, point the analyzer at the artifact instead of rebuilding.
- **Offline analysis.** When the emulator is unavailable, capture DroidBot output on another machine and feed it in.

## Behaviour

- Reads `gradle/libs.versions.toml` from `--repo` and resolves `--dependency` to one or more Kotlin package roots.
- Materialises BEFORE and AFTER shadow copies under `--output-dir/phase1/`.
- Runs Phase 2 (static). Runs Phase 3 (dynamic) unless `--skip-dynamic` is passed.
- Consolidates into `phase4/consolidated.json` and renders the HTML report into `<output-dir>/report/`.

## Example

```bash
kmp-impact analyze \
  --repo ~/projects/pokedex-kmp \
  --dependency io.ktor \
  --before-version 2.3.8 \
  --after-version 2.3.11 \
  --output-dir output \
  --skip-dynamic
open output/report/index.html
```

## See also

- [Guides → Analyzing a bump locally](../../guides/analyzing-locally.md)
- [`run-scenario`](run-scenario.md) — for repeatable, scenario-driven runs.
- [`AnalysisConfig`](../contracts/analysis-config.md) — the config object built from these flags.
