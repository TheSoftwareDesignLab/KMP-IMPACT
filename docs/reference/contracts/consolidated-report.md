# `ConsolidatedReport`

The Phase 4 canonical artifact. Consumed by the HTML report renderer and the PR-comment composer. Written to `phase4/consolidated.json`.

## Fields

| Field | Type | Notes |
|---|---|---|
| `bump` | `BumpRef` | The dependency change. |
| `static` | `StaticBlock` | Wraps `impact_graph` and aggregate counters. |
| `dynamic` | `DynamicBlock` | Wraps `ui_diff` and a `status` summary. |
| `traceability` | `list[TraceabilityRow]` | Best-effort file → screen mapping by package proximity. |
| `risk` | `str` | `LOW`, `MEDIUM`, or `HIGH`. |
| `recommendation` | `str` | One-sentence human-readable summary. |

The risk label is a review-prioritization cue, not a validated failure predictor. The function lives in [`phase4_consolidate/scorer.py`](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/blob/main/src/kmp_impact_analyzer/phase4_consolidate/scorer.py) — changes to the heuristic must come with new fixtures under `tests/`.

## Example

```json
{
  "bump": {
    "dependency": "io.ktor",
    "before": "2.3.8",
    "after": "2.3.11",
    "category": "library_kotlin"
  },
  "static": {
    "impact_graph": { /* ImpactGraph */ }
  },
  "dynamic": {
    "ui_diff": { /* UIDiffResult */ }
  },
  "traceability": [
    {
      "file": "shared/src/commonMain/kotlin/com/example/api/PokedexClient.kt",
      "screens": ["PokemonDetailScreen"]
    }
  ],
  "risk": "LOW",
  "recommendation": "Safe to merge after CI."
}
```

## Related

- [`ImpactGraph`](impact-graph.md)
- [`UIDiffResult`](ui-diff-result.md)
- [Architecture → Phase 4](../../architecture/phase-4-consolidation.md)
- [Architecture → Phase 5](../../architecture/phase-5-visualization.md) — how this object is rendered.
