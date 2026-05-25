# `kmp-impact run-scenario`

Run a YAML-defined scenario. Scenarios bundle one or more bumps with an optional ground truth and are the recommended way to benchmark changes to the analyzer.

## Synopsis

```bash
kmp-impact run-scenario \
  --scenario-dir DIR \
  --output-dir OUT \
  [--skip-dynamic]
```

## Flags

| Flag | Required | Description |
|---|---|---|
| `--scenario-dir` | yes | Path to a scenario directory containing `scenario.yml`. |
| `--output-dir` | yes | Destination for `phaseN/` and the HTML report. |
| `--skip-dynamic` | no | Skip Phase 3 for every bump in the scenario. |

## Scenario directory layout

```text
scenario/
├── scenario.yml         # bumps and metadata
├── ground_truth.yml     # expected file/screen impact (optional)
└── README.md            # human description
```

See [Guides → Writing Scenarios](../../guides/scenarios.md) for the file format and authoring workflow.

## Example

```bash
kmp-impact run-scenario \
  --scenario-dir scenarios/pokedex_ktor_minor \
  --output-dir output-pokedex \
  --skip-dynamic
```

When `ground_truth.yml` is present, the report includes a metrics card with Precision / Recall / F1 for both files and screens.

## See also

- [`analyze`](analyze.md) — for a single ad-hoc bump.
- [`evaluate`](evaluate.md) — for offline metric computation against an existing result.
