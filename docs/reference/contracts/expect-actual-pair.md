# `ExpectActualPair`

A detected `expect` declaration together with all the `actual` files that implement it. Surfaced as a review target — **not** a compatibility proof. The analyzer does not verify that every required target has an `actual` implementation.

## Fields

| Field | Type | Default | Notes |
|---|---|---|---|
| `expect_fqcn` | `str` | — | Fully qualified name of the `expect` declaration (e.g. `com.example.HttpEngineFactory`). |
| `expect_file` | `str` | — | Path to the `.kt` file that contains the `expect` declaration (typically under `commonMain`). |
| `actual_files` | `list[str]` | `[]` | Paths of the `.kt` files that contain matching `actual` declarations across platform source sets. |

## Example

```json
{
  "expect_fqcn": "com.example.HttpEngineFactory",
  "expect_file": "shared/src/commonMain/kotlin/com/example/HttpEngineFactory.kt",
  "actual_files": [
    "shared/src/androidMain/kotlin/com/example/HttpEngineFactory.android.kt",
    "shared/src/iosMain/kotlin/com/example/HttpEngineFactory.ios.kt"
  ]
}
```

## Where it appears

Inside [`ImpactGraph.expect_actual_pairs`](impact-graph.md). The list is populated only with pairs whose `expect_file` is impacted (direct or transitive) by the current bump. Pairs that touch unaffected declarations are not emitted.

## What it does **not** guarantee

- It does **not** check that every required target (Android, iOS, Desktop, …) has an `actual` implementation. The analyzer reports what it sees in the source tree.
- It does **not** verify that the signature of `actual` matches the signature of `expect`. Kotlin's compiler enforces that; the analyzer reads the source tree only.
- It does **not** flag dangling `actual` declarations whose `expect` is missing.

These are deliberate limitations of a parse-only analyzer; see [L8 — DI](../../troubleshooting.md#l8-dependency-injection-misses) for the general pattern of "we only see imports and declarations".

## See also

- [`FileImpact`](file-impact.md) — the per-file record. `relation = "expect_actual"` is used when a file is impacted *because* it participates in a pair.
- [Architecture → Phase 2](../../architecture/phase-2-static-analysis.md)
