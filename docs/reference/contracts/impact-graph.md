# `ImpactGraph`

The aggregate Phase 2 output. Written to `phase2/impact_graph.json` and consumed by Phase 4.

## Fields

| Field | Type | Notes |
|---|---|---|
| `files` | `list[FileImpact]` | One entry per impacted Kotlin file. |
| `counts` | `ImpactCounts` | Pre-computed aggregates. |
| `bump` | `BumpRef` | The dependency change this graph belongs to. |

## `ImpactCounts`

| Field | Type | Notes |
|---|---|---|
| `direct` | `int` | Count of `impact_level == 2` files. |
| `transitive` | `int` | Count of `impact_level == 1` files. |
| `total` | `int` | `direct + transitive`. |
| `by_source_set` | `dict[str, int]` | Per-source-set total — drives the sunburst preview. |
| `expect_actual_pairs` | `int` | Total `ExpectActualHit` entries across all files. |

## `BumpRef`

| Field | Type | Notes |
|---|---|---|
| `dependency` | `str` | Maven group or coordinate. |
| `before` | `str` | BEFORE version. |
| `after` | `str` | AFTER version. |
| `category` | `str` | `library_kotlin`, `plugin_or_toolchain`, or `unknown`. |

## Example

```json
{
  "files": [ /* FileImpact[] */ ],
  "counts": {
    "direct": 2,
    "transitive": 5,
    "total": 7,
    "by_source_set": {
      "commonMain": 4,
      "androidMain": 2,
      "iosMain": 1
    },
    "expect_actual_pairs": 1
  },
  "bump": {
    "dependency": "io.ktor",
    "before": "2.3.8",
    "after": "2.3.11",
    "category": "library_kotlin"
  }
}
```

## Related

- [`FileImpact`](file-impact.md)
- [Architecture → Phase 2](../../architecture/phase-2-static-analysis.md)
