# `ImpactGraph`

Phase 2 aggregate output. Written to `phase2/impact_graph.json` and consumed by Phase 4.

## Fields

| Field | Type | Default | Notes |
|---|---|---|---|
| `dependency_group` | `str` | — | Maven group of the bump being analysed. |
| `version_before` | `str` | — | BEFORE version. |
| `version_after` | `str` | — | AFTER version. |
| `seed_files` | `list[str]` | `[]` | Files that *directly* import the bumped dependency. Every `FileImpact` with `relation = "direct"` corresponds to an entry here. |
| `impacted_files` | `list[FileImpact]` | `[]` | One entry per impacted Kotlin file. |
| `expect_actual_pairs` | `list[ExpectActualPair]` | `[]` | Detected `expect`/`actual` pairs whose `expect` file is impacted. |
| `total_project_files` | `int` | `0` | Total `.kt` files scanned (denominator for coverage statistics). |
| `total_impacted` | `int` | `0` | `len(impacted_files)`. Persisted to avoid recomputing in downstream consumers. |
| `ksp_edges_added` | `int` | `0` | Extra edges contributed by the KSP-driven symbol index, if KSP output is available. `0` when the analyzer falls back to the Tree-sitter-only mode. |

## Example

```json
{
  "dependency_group": "io.ktor",
  "version_before": "2.3.8",
  "version_after": "2.3.11",
  "seed_files": [
    "shared/src/commonMain/kotlin/com/example/api/PokedexClient.kt"
  ],
  "impacted_files": [ /* FileImpact[] */ ],
  "expect_actual_pairs": [ /* ExpectActualPair[] */ ],
  "total_project_files": 85,
  "total_impacted": 7,
  "ksp_edges_added": 12
}
```

## Aggregating per source set

The model does not pre-compute counters per source set; the renderer iterates `impacted_files` and groups by `source_set` to build the sunburst preview. Consumers that need per-source-set counts can do the same:

```python
from collections import Counter

graph = ImpactGraph.model_validate_json(open("output/phase2/impact_graph.json").read())
by_source_set = Counter(f.source_set for f in graph.impacted_files)
```

## Related

- [`FileImpact`](file-impact.md)
- [`ExpectActualPair`](expect-actual-pair.md)
- [Architecture → Phase 2](../../architecture/phase-2-static-analysis.md)
