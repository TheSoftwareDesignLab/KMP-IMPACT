# `kmp-impact detect-version-changes`

Diff two `libs.versions.toml` files. Used by the GitHub Action's `detect` job to feed `--dependency`, `--before-version`, and `--after-version` automatically.

## Synopsis

```bash
kmp-impact detect-version-changes \
  --before /tmp/base.libs.versions.toml \
  --after  /tmp/head.libs.versions.toml
```

## Flags

| Flag | Required | Description |
|---|---|---|
| `--before` | yes | Path to the BEFORE catalog (typically PR merge-base). |
| `--after` | yes | Path to the AFTER catalog (typically PR head). |

## Output

JSON on stdout. One entry per detected change:

```json
[
  {
    "alias": "ktor",
    "maven": "io.ktor",
    "before": "2.3.8",
    "after": "2.3.11",
    "category": "library_kotlin"
  }
]
```

If the diff is empty, stdout is `[]` and the exit code is `0`.

## Categories

| Category | Examples | Pipeline behaviour |
|---|---|---|
| `library_kotlin` | `io.ktor`, `io.insert-koin`, `org.jetbrains.kotlinx`, `com.squareup.okhttp3` | Full static + dynamic analysis. |
| `plugin_or_toolchain` | `com.android.application`, `org.jetbrains.kotlin`, `com.google.devtools.ksp`, `gradle-wrapper` | Static impact reported as zero by design ([L2](../../troubleshooting.md#l2-plugin_or_toolchain-bumps-report-zero-static-impact)). |
| `unknown` | Coordinate not present in the Maven → Kotlin map. | Analysis runs; Precision may be conservative. |

## See also

- [Guides → Configuring Dependabot](../../guides/dependabot.md) — `category`-aware ignore rules.
- [Architecture → Phase 1](../../architecture/phase-1-shadow-build.md) — how the bump is resolved against the catalog.
