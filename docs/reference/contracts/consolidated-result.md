# `ConsolidatedResult`

The Phase 4 canonical artifact. Consumed by the HTML report renderer and the PR-comment composer. Written to `phase4/consolidated.json`.

## Fields

| Field | Type | Default | Notes |
|---|---|---|---|
| `dependency_group` | `str` | — | Maven group of the bump. |
| `version_before` | `str` | — | BEFORE version. |
| `version_after` | `str` | — | AFTER version. |
| `static_impact` | `ImpactGraph` | — | Full Phase 2 output. |
| `dynamic_regressions` | `UIRegressions` | — | Full Phase 3 output. |
| `screen_mappings` | `list[ScreenMapping]` | `[]` | Best-effort file → screen mappings produced by Phase 4's heuristic. |
| `trace` | `list[TraceEntry]` | `[]` | Flattened file-by-file traceability rows used by the HTML *Traceability* tab. |
| `impacted_screens` | `list[str]` | `[]` | Distinct screen names reached by at least one impacted file. |
| `total_impacted_files` | `int` | `0` | Cached `len(static_impact.impacted_files)`. |
| `total_impacted_screens` | `int` | `0` | Cached `len(impacted_screens)`. |
| `stack_compatibility` | `StackCompatibilityReport` | empty | Warnings about Kotlin / AGP / Compose vs. the bumped dependency. |

The risk label and the one-sentence recommendation are **not** persisted here — the HTML renderer computes them at report time from these counters and the bump category.

## `ScreenMapping`

| Field | Type | Notes |
|---|---|---|
| `screen_name` | `str` | Identifier of the screen. |
| `mapped_files` | `list[str]` | Files that the heuristic associated with the screen. |
| `confidence` | `float` | `[0.0, 1.0]`. Driven by package proximity and import overlap. |
| `method` | `str` | Short tag describing how the mapping was inferred (`package_match`, `import_overlap`, …). |

## `TraceEntry`

| Field | Type | Notes |
|---|---|---|
| `file_path` | `str` | Same as `FileImpact.file_path`. |
| `relation` | `ImpactRelation` | `direct`, `transitive`, or `expect_actual`. |
| `distance` | `int` | BFS distance from the nearest seed. |
| `screens` | `list[str]` | Screens this file is associated with. |
| `metrics` | `SourceMetrics` | Same as `FileImpact.metrics`. |

## `StackCompatibilityReport`

```json
{
  "warnings": [
    {
      "severity": "warning",
      "title": "Compose Compiler plugin missing under Kotlin 2.x",
      "detail": "...",
      "suggestion": "Apply org.jetbrains.kotlin.plugin.compose to every Compose module.",
      "detected": { "kotlin": "2.0.21" },
      "required": { "kotlin.plugin.compose": "applied" }
    }
  ],
  "detected": { "kotlin": "2.0.21", "agp": "8.5.0", "compose": "1.6.10" }
}
```

`severity` is `info`, `warning`, or `error`. The HTML report surfaces `warning` and `error` entries above the Summary tab so reviewers see them immediately.

## Example (abridged)

```json
{
  "dependency_group": "io.ktor",
  "version_before": "2.3.8",
  "version_after": "2.3.11",
  "static_impact": { /* ImpactGraph */ },
  "dynamic_regressions": { /* UIRegressions */ },
  "screen_mappings": [
    {
      "screen_name": "PokemonDetailScreen",
      "mapped_files": ["shared/src/commonMain/kotlin/com/example/api/PokedexClient.kt"],
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

## Related

- [`ImpactGraph`](impact-graph.md)
- [`UIRegressions`](ui-regressions.md)
- [Architecture → Phase 4](../../architecture/phase-4-consolidation.md)
- [Architecture → Phase 5](../../architecture/phase-5-visualization.md) — how this object becomes the HTML report.
