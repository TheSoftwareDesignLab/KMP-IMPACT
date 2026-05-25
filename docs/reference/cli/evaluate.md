# `kmp-impact evaluate`

Compare a pipeline result against a ground truth and emit Precision / Recall / F1 for both files and screens.

## Synopsis

```bash
kmp-impact evaluate \
  --results CONSOLIDATED.json \
  --ground-truth GROUND_TRUTH.yml \
  [--output-dir OUT]
```

## Flags

| Flag | Required | Default | Description |
|---|---|---|---|
| `--results` | yes | — | Path to `phase4/consolidated.json` from a previous pipeline run. |
| `--ground-truth` | yes | — | Path to a `ground_truth.yml` with the expected impacted files and screens. |
| `--output-dir` | no | `output/evaluation` | Directory where the per-scenario evaluation artefacts are written. |

## Behaviour

1. Loads the `ConsolidatedResult` model from `--results`.
2. Reads the expected sets from `--ground-truth`.
3. Computes per-file and per-screen Precision / Recall / F1.
4. Writes a small report under `--output-dir` and logs the headline metrics.

A log line summarises the result:

```text
F1=0.79  Precision=0.83  Recall=0.75
```

The on-disk artefact is an `EvaluationResult` JSON with the full true-positive, false-positive, and false-negative file/screen lists for inspection.

## Metric definitions

Given the analyzer's set `A` and the ground-truth set `M`:

```text
Precision = |A ∩ M| / |A|
Recall    = |A ∩ M| / |M|
F1        = 2 · P · R / (P + R)
```

## Example

```bash
kmp-impact evaluate \
  --results output/phase4/consolidated.json \
  --ground-truth scenarios/pokedex_ktor_minor/ground_truth.yml \
  --output-dir output/evaluation
```

## See also

- [Guides → Writing Scenarios](../../guides/scenarios.md) — how to author `ground_truth.yml`.
- [`ConsolidatedResult`](../contracts/consolidated-result.md) — the input contract.
