# `kmp-impact evaluate`

Compare a pipeline result against a ground truth and emit Precision / Recall / F1. Used to score scenarios and to validate analyzer changes against a known-good baseline.

## Synopsis

```bash
kmp-impact evaluate \
  --results output/phase4/consolidated.json \
  --ground-truth scenarios/cursokmpapp_ktor/ground_truth.yml
```

## Flags

| Flag | Required | Description |
|---|---|---|
| `--results` | yes | Path to `phase4/consolidated.json` from a previous pipeline run. |
| `--ground-truth` | yes | Path to a `ground_truth.yml` with `direct_files`, `transitive_files`, and `ui_screens`. |

## Output

JSON on stdout. Same shape as the metrics card embedded in the HTML report:

```json
{
  "files": { "precision": 0.83, "recall": 0.75, "f1": 0.79 },
  "screens": { "precision": 1.00, "recall": 1.00, "f1": 1.00 }
}
```

## Metric definitions

Given the analyzer's set `A` and the ground-truth set `M`:

```text
Precision = |A ∩ M| / |A|
Recall    = |A ∩ M| / |M|
F1        = 2 · P · R / (P + R)
```

A bump where both `A` and `M` are empty is reported with `null` metrics — it does not contribute to mean values.

## See also

- [Guides → Writing Scenarios](../../guides/scenarios.md) — how to author `ground_truth.yml`.
- [`run-scenario`](run-scenario.md) — the scenario runner that embeds these metrics automatically.
