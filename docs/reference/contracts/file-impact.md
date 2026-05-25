# `FileImpact`

The unit of Phase 2 output. One entry per impacted Kotlin file.

## Fields

| Field | Type | Notes |
|---|---|---|
| `path` | `str` | File path relative to the project root. |
| `source_set` | `str` | `commonMain`, `androidMain`, `iosMain`, `desktopMain`, `commonTest`, … `unknown` when the file does not live under a `src/<sourceSet>Main/kotlin/` path. |
| `impact_level` | `0 \| 1 \| 2` | 2 = direct (matches the bumped library's package root), 1 = transitive (reached by BFS from a direct file), 0 = not impacted (rarely persisted). |
| `imports_matched` | `list[str]` | Fully qualified imports that triggered the direct match. Empty for transitive files. |
| `propagated_from` | `list[str]` | Path of the parent file in the BFS, per propagation level. Empty for direct files. |
| `expect_actual` | `list[ExpectActualHit]` | Detected `expect`/`actual` declarations in this file. |
| `metrics` | `FileMetrics` | `rloc`, `mcc`. |

## Example

```json
{
  "path": "shared/src/commonMain/kotlin/com/example/api/PokedexClient.kt",
  "source_set": "commonMain",
  "impact_level": 2,
  "imports_matched": ["io.ktor.client.HttpClient"],
  "propagated_from": [],
  "expect_actual": [
    { "kind": "class", "name": "HttpEngineFactory", "role": "expect" }
  ],
  "metrics": { "rloc": 84, "mcc": 12 }
}
```

## Source-set semantics

`source_set` is derived from the file path. The analyzer recognises the conventional KMP layout `src/<sourceSet>Main/kotlin/`. Files outside this convention receive `source_set = "unknown"` — see [L3](../../troubleshooting.md#l3-non-convention-source-set-layout).

## Related

- [`ExpectActualHit`](expect-actual-hit.md)
- [`ImpactGraph`](impact-graph.md) — the aggregate that contains `FileImpact[]`.
- [Architecture → Phase 2](../../architecture/phase-2-static-analysis.md)
