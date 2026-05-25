# `UIRegressions`

Phase 3 dynamic output. Comparison of the BEFORE and AFTER UI Transition Graphs collected by DroidBot.

## Fields

| Field | Type | Default | Notes |
|---|---|---|---|
| `status` | `DynamicStatus` | `skipped` | See enum below. |
| `blocked_reason` | `str` | `""` | Filled when `status = "blocked"`. Empty otherwise. |
| `before_screens` | `list[str]` | `[]` | Distinct screen identifiers observed in the BEFORE UTG. |
| `after_screens` | `list[str]` | `[]` | Distinct screen identifiers observed in the AFTER UTG. |
| `diffs` | `list[ScreenDiff]` | `[]` | Per-screen diff entries. See below. |
| `activity_coverage_before` | `dict[str, int]` | `{}` | Activity name → visit count, BEFORE state. |
| `activity_coverage_after` | `dict[str, int]` | `{}` | Activity name → visit count, AFTER state. |
| `nodes_before` | `int` | `0` | Total UTG nodes in BEFORE. |
| `nodes_after` | `int` | `0` | Total UTG nodes in AFTER. |
| `structures_before` | `int` | `0` | Distinct view-tree structures in BEFORE (deduplicated by `structure_id`). |
| `structures_after` | `int` | `0` | Same for AFTER. |

## `DynamicStatus` enum { #dynamicstatus }

| Value | Meaning |
|---|---|
| `completed` | Both APKs built and DroidBot returned UTGs. |
| `blocked` | Build failed or DroidBot produced no UTG. Paired with `blocked_reason`. |
| `skipped` | `--skip-dynamic` was passed, or no Android module could be detected. |

## `ScreenDiff`

| Field | Type | Notes |
|---|---|---|
| `screen_name` | `str` | Identifier of the screen. |
| `status` | `str` | `missing` (in BEFORE only), `new` (in AFTER only), or `changed` (in both, with internal differences). |
| `details` | `str` | Human-readable description of the difference. |

## Examples

`completed` run with no diffs:

```json
{
  "status": "completed",
  "blocked_reason": "",
  "before_screens": ["MainActivity", "DetailActivity"],
  "after_screens":  ["MainActivity", "DetailActivity"],
  "diffs": [],
  "activity_coverage_before": { "MainActivity": 18, "DetailActivity": 6 },
  "activity_coverage_after":  { "MainActivity": 17, "DetailActivity": 7 },
  "nodes_before": 14, "nodes_after": 14,
  "structures_before": 6, "structures_after": 6
}
```

`blocked` run:

```json
{
  "status": "blocked",
  "blocked_reason": "DroidBot produced no UTG artifact",
  "before_screens": [],
  "after_screens": [],
  "diffs": []
}
```

`skipped` run (`--skip-dynamic`):

```json
{ "status": "skipped", "blocked_reason": "", "before_screens": [], "after_screens": [], "diffs": [] }
```

## Related

- [Architecture → Phase 3](../../architecture/phase-3-dynamic-analysis.md)
- [Guides → Diagnosing a BLOCKED phase](../../guides/diagnosing-blocked.md)
