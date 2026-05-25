# `kmp-impact analyze`

Run the full pipeline against a single dependency bump.

## Synopsis

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

## Flags

| Flag | Required | Description |
|---|---|---|
| `--repo` | yes | Path to the KMP project to analyse. |
| `--dependency` | yes | Maven group or coordinate substring (e.g. `io.ktor`, `org.jetbrains.compose`). |
| `--before-version` | yes | Version to consider as BEFORE. |
| `--after-version` | yes | Version to consider as AFTER. |
| `--output-dir` | yes | Directory where shadow copies, phase outputs, and the HTML report are written. |
| `--skip-dynamic` | no | Skip Phase 3. The dynamic tab in the report shows `BLOCKED — skipped by user`. |
| `--keep-shadows` | no | Keep `phase1/before/` and `phase1/after/` after the run. Useful for re-running a single phase. |

## Behaviour

- Reads `gradle/libs.versions.toml` from `--repo` and resolves `--dependency` to one or more Kotlin package roots.
- Materialises BEFORE and AFTER shadow copies under `--output-dir/phase1/`.
- Runs Phase 2 (static), and Phase 3 (dynamic) unless `--skip-dynamic` is passed.
- Consolidates into `phase4/consolidated.json` and renders the HTML report into `report/`.

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
- [`run-scenario`](run-scenario.md) — for repeatable, ground-truth-driven runs.
