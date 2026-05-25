# `UIDiffResult`

The Phase 3 output. Comparison of the BEFORE and AFTER UI Transition Graphs.

## Fields

| Field | Type | Notes |
|---|---|---|
| `status` | `DynamicStatus` | See enum below. |
| `before_screens` | `int` | Distinct states observed in the BEFORE UTG. |
| `after_screens` | `int` | Distinct states observed in the AFTER UTG. |
| `new_screens` | `list[ScreenRef]` | States present in AFTER but not BEFORE. |
| `missing_screens` | `list[ScreenRef]` | States present in BEFORE but not AFTER. |
| `new_edges` | `list[EdgeRef]` | Transitions added. |
| `missing_edges` | `list[EdgeRef]` | Transitions removed. |
| `blocked_reason` | `str \| null` | Set when `status = "BLOCKED"`. |

## `DynamicStatus` { #dynamicstatus }

A small enum:

| Value | Meaning |
|---|---|
| `OK` | Both APKs built and DroidBot returned UTGs. |
| `BLOCKED` | Build failed or DroidBot produced no UTG. Paired with `blocked_reason`. |
| `SKIPPED_BY_USER` | `--skip-dynamic` was passed. |

## Examples

`OK` case:

```json
{
  "status": "OK",
  "before_screens": 14,
  "after_screens": 14,
  "new_screens": [],
  "missing_screens": [],
  "new_edges": [],
  "missing_edges": [],
  "blocked_reason": null
}
```

`BLOCKED` case:

```json
{
  "status": "BLOCKED",
  "blocked_reason": "DroidBot produced no UTG artifact"
}
```

## Related

- [Architecture → Phase 3](../../architecture/phase-3-dynamic-analysis.md)
- [Guides → Diagnosing a BLOCKED phase](../../guides/diagnosing-blocked.md)
