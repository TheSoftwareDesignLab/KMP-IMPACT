# `FileImpact`

One impacted Kotlin file. Emitted by Phase 2 and embedded in `ImpactGraph.impacted_files`.

## Fields

| Field | Type | Default | Notes |
|---|---|---|---|
| `file_path` | `str` | — | File path relative to the project root. |
| `relation` | `ImpactRelation` | — | `direct`, `transitive`, or `expect_actual`. |
| `distance` | `int` | `0` | BFS distance from the nearest direct seed. `0` for direct files. |
| `imports_from_dependency` | `list[str]` | `[]` | Fully qualified imports from the bumped dependency that triggered the direct match. Empty for transitive files. |
| `propagated_from` | `list[str]` | `[]` | Parent files in the BFS, per propagation level. Empty for direct files. |
| `metrics` | `SourceMetrics` | defaults | Per-file metrics — see below. |
| `declarations` | `list[str]` | `[]` | Names of declarations in this file that are touched by the impact path. |
| `source_set` | `str` | `"common"` | `common`, `android`, `ios`, `desktop`, `commonTest`, … `unknown` when the file does not live under a `src/<sourceSet>Main/kotlin/` path. |

## `ImpactRelation` enum

| Value | Meaning |
|---|---|
| `direct` | The file imports a symbol from the bumped dependency. |
| `transitive` | The file is reached by BFS from at least one direct file. |
| `expect_actual` | The file participates in an `expect` / `actual` pair touched by the change. |

## Computed convenience: `impact_level`

```python
file.impact_level  # 2 if relation == direct, 1 otherwise
```

This is a Python `@property`, not a persisted field. The HTML renderer and the CodeCharta exporter both reuse it. Consumers reading the JSON directly should look at `relation` instead.

## `SourceMetrics`

| Field | Type | Default | Notes |
|---|---|---|---|
| `rloc` | `int` | `0` | Real lines of code. Drives building **area** in CodeCharta. |
| `functions` | `int` | `0` | Function count. |
| `mcc` | `int` | `1` | McCabe-like complexity heuristic. Drives building **height** in CodeCharta. |
| `detekt_findings` | `int` | `0` | Count of Detekt findings when a report is provided. |
| `test_coverage` | `float` | `-1.0` | Kover coverage if available. `-1.0` means not measured. |
| `metrics_source` | `str` | `"heuristic"` | `heuristic` or `detekt` — whether `mcc` was computed by the bundled heuristic or sourced from a Detekt report. |

## Example

```json
{
  "file_path": "shared/src/commonMain/kotlin/com/example/api/PokedexClient.kt",
  "relation": "direct",
  "distance": 0,
  "imports_from_dependency": [
    "io.ktor.client.HttpClient",
    "io.ktor.client.request.get"
  ],
  "propagated_from": [],
  "metrics": {
    "rloc": 84,
    "functions": 9,
    "mcc": 12,
    "detekt_findings": 0,
    "test_coverage": -1.0,
    "metrics_source": "heuristic"
  },
  "declarations": ["PokedexClient", "fetchPokemonList"],
  "source_set": "common"
}
```

## Source-set semantics

`source_set` is derived from the file path. The analyzer recognises the conventional KMP layout `src/<sourceSet>Main/kotlin/` and `src/<sourceSet>Test/kotlin/`. Files outside this convention receive `source_set = "unknown"` — see [L3](../../troubleshooting.md#l3-non-convention-source-set-layout).

The string is the *short* source-set name (`common`, `android`, `ios`, `desktop`, `commonTest`), not the qualified Gradle name (`commonMain`, `androidMain`).

## Related

- [`ExpectActualPair`](expect-actual-pair.md) — pairs are listed separately on `ImpactGraph.expect_actual_pairs`, not embedded in `FileImpact`.
- [`ImpactGraph`](impact-graph.md) — the aggregate that owns `FileImpact[]`.
- [Architecture → Phase 2](../../architecture/phase-2-static-analysis.md)
