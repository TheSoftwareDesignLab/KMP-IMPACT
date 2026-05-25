# Phase 4 — Consolidation

The fourth phase produces the single canonical artifact `phase4/consolidated.json` that the HTML report and the PR comment consume.

## Inputs

- `phase1/manifest.json` — the bump and category.
- `phase2/impact_graph.json` — static impact and source-set distribution.
- `phase3/ui_regressions.json` — dynamic impact, when available.

## Outputs

```text
phase4/
└── consolidated.json
```

## What happens

1. Load all three input artifacts and validate them against their Pydantic schemas.
2. Compute aggregate counters across phases (direct vs. transitive, by source set, `expect`/`actual` pairs).
3. Build the **traceability table** — best-effort mapping of impacted files to UI screens by package proximity. This is a reviewer aid, not a coverage proof.
4. Compute the **risk label** with `phase4_consolidate/scorer.py`. The label maps `(direct_count, transitive_count, ui_diff_count, bump.category)` to `LOW`, `MEDIUM`, or `HIGH`. It is a review-prioritization cue, not a validated failure predictor.
5. Compose a one-sentence **recommendation** that matches the risk label.

## Output sample (abridged)

```json
{
  "dependency_group": "io.ktor",
  "version_before": "2.3.8",
  "version_after": "2.3.11",
  "static_impact":       { /* ImpactGraph */ },
  "dynamic_regressions": { /* UIRegressions */ },
  "screen_mappings": [
    {
      "screen_name": "PokemonDetailScreen",
      "mapped_files": ["shared/src/commonMain/kotlin/.../PokedexClient.kt"],
      "confidence": 0.78,
      "method": "package_match"
    }
  ],
  "trace": [ /* TraceEntry[] */ ],
  "impacted_screens": ["PokemonDetailScreen"],
  "total_impacted_files": 7,
  "total_impacted_screens": 1,
  "stack_compatibility": { "warnings": [], "detected": { "kotlin": "2.0.21" } }
}
```

The risk label and the one-sentence recommendation are computed by the HTML renderer from these counters and the bump category — they are not persisted in the JSON.

## Edge cases

- **`BLOCKED` Phase 3.** The consolidator still runs and emits `dynamic.ui_diff.status = "BLOCKED"` with the `blocked_reason` propagated. The risk label can still be computed from the static counters.
- **Empty static impact.** Plugin/toolchain bumps land here. The risk label falls to `LOW` unless the dynamic phase shows screen diffs.
- **Empty dynamic impact.** Common for patch/minor bumps. The label is computed from static counters alone.

## Where the risk function lives

`src/kmp_impact_analyzer/phase4_consolidate/scorer.py`. Changes to the heuristic must come with new fixtures under `tests/`.

## Contract

- [`ConsolidatedResult`](../reference/contracts/consolidated-result.md)

## Next phase

Phase 5 reads `consolidated.json` and produces the CodeCharta JSON and the HTML report.

→ [Phase 5 — Visualization](phase-5-visualization.md)
