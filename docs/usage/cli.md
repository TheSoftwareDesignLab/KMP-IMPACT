# CLI reference

KMP-IMPACT ships a single entry point — `kmp-impact` — with four subcommands.

```bash
kmp-impact [COMMAND] [OPTIONS]
```

## `analyze`

Run the full pipeline against a single dependency bump.

```bash
kmp-impact analyze \
  --repo PATH \
  --dependency NAME \
  --before-version A \
  --after-version B \
  --output-dir OUT \
  [--skip-dynamic] \
  [--keep-shadows]
```

| Flag | Required | Description |
|---|---|---|
| `--repo` | yes | Path to the KMP project to analyse. |
| `--dependency` | yes | Maven group or coordinate substring (e.g. `io.ktor`, `org.jetbrains.compose`). |
| `--before-version` | yes | Version to consider as BEFORE. |
| `--after-version` | yes | Version to consider as AFTER. |
| `--output-dir` | yes | Directory where shadow copies, phase outputs and the HTML report are written. |
| `--skip-dynamic` | no | Skip the DroidBot phase. The dynamic tab in the report shows `BLOCKED — skipped by user`. |
| `--keep-shadows` | no | Keep `phase1/before/` and `phase1/after/` after the run. Useful for debugging. |

## `run-scenario`

Run a YAML-defined scenario. Scenarios bundle one or more bumps with an optional ground truth.

```bash
kmp-impact run-scenario \
  --scenario-dir DIR \
  --output-dir OUT \
  [--skip-dynamic]
```

A scenario directory contains:

```text
scenario/
├── scenario.yml         # bumps and metadata
├── ground_truth.yml     # expected file/screen impact (optional)
└── README.md            # human description
```

See [Scenarios](scenarios.md) for the file format.

## `detect-version-changes`

Diff two `libs.versions.toml` files. Used by the GitHub Action to feed `--dependency`, `--before-version` and `--after-version` automatically.

```bash
kmp-impact detect-version-changes \
  --before /tmp/base.libs.versions.toml \
  --after /tmp/head.libs.versions.toml
```

Output is JSON on stdout:

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

Categories:

| Category | Example |
|---|---|
| `library_kotlin` | Regular Kotlin/JVM library bumps — analysed end-to-end. |
| `plugin_or_toolchain` | AGP, KSP, Gradle wrapper bumps — static impact is reported as zero **by design** (the change lives outside Kotlin sources). |
| `unknown` | Coordinate not present in the Maven → Kotlin import map. The analysis still runs but `Precision` for this bump will be conservative. |

## `evaluate`

Compare a pipeline result against a ground truth and emit Precision / Recall / F1.

```bash
kmp-impact evaluate \
  --results output/phase4/consolidated.json \
  --ground-truth scenarios/cursokmpapp_ktor/ground_truth.yml
```

The metrics are also embedded in `output/report/index.html` when a `ground_truth.yml` is present in the scenario.

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Pipeline produced a report. The report may still mark phases as `BLOCKED`. |
| `1` | CLI usage error (missing flag, invalid path). |
| `2` | Unrecoverable pipeline error (uncaught exception). Stack trace on stderr. |

A `BLOCKED` phase is **not** an error: it means the analyzer correctly detected that it could not produce real data for that phase and refused to fabricate one.
