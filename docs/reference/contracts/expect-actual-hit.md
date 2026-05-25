# `ExpectActualHit`

A detected `expect` or `actual` declaration in an impacted file. Surfaces are review targets — **not** compatibility proofs. The analyzer does not verify that every `expect` has matching `actual` implementations across required targets.

## Fields

| Field | Type | Values |
|---|---|---|
| `kind` | `str` | `class`, `object`, `function`, `property`. |
| `name` | `str` | The simple name of the declaration. |
| `role` | `str` | `expect` or `actual`. |

## Example

```json
{ "kind": "class", "name": "HttpEngineFactory", "role": "expect" }
```

```json
{ "kind": "function", "name": "platformName", "role": "actual" }
```

## Where it appears

Inside [`FileImpact.expect_actual`](file-impact.md) — one list per impacted file. Aggregate counters per source set are exposed in [`ImpactGraph.counts.expect_actual_pairs`](impact-graph.md).

## Related

- [Architecture → Phase 2](../../architecture/phase-2-static-analysis.md) — how detection works.
- [Guides → Reading the Report](../../guides/reading-the-report.md) — how the count shows up in the PR comment.
