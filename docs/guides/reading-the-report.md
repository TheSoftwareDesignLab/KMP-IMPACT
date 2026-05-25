# Reading the Report

Every successful run produces a self-contained HTML report at `output/report/index.html`. In CI it is also published to GitHub Pages and linked from the PR comment.

## PR comment payload

The comment is the triage entry point. It restates the bump, the risk label, and the headline counters:

```text
KMP-IMPACT analysis · io.ktor 2.3.8 -> 2.3.11

Static impact      : 7 files (2 direct, 5 transitive)
Source sets        : commonMain(5), androidMain(1), iosMain(1)
expect/actual      : 1 pair touched
Dynamic impact     : 0 new screens, 0 missing screens
Risk label         : LOW
Recommendation     : safe to merge after CI

Full report   -> https://<owner>.github.io/<repo>/pr-42/
Raw artifacts -> workflow run #123
```

The **risk label** is a review-prioritization cue, not a validated failure predictor. The recommendation is generated from the counters and the bump category — it complements reviewer judgment; it does not replace it.

## HTML report — six tabs

| Tab | What it shows |
|---|---|
| **Summary** | Bump, impact counts, source-set distribution sunburst, risk label, recommendation. |
| **Static impact** | List of `.kt` files marked direct (`level 2`) or transitive (`level 1`), with the propagation tree (BFS edges persisted from `propagated_from`). |
| **UI impact** | DroidBot UTG diff: new screens, missing screens, new edges, missing edges. Empty when `--skip-dynamic` or `BLOCKED`. |
| **Traceability** | File → Screen mapping with sticky header. Best-effort match by package proximity. |
| **Visualization** | Interactive CodeCharta viewer (`area=rloc`, `height=mcc`, `color=impact_level`) plus a downloadable JSON. |
| **Raw artifacts** | Links to every phase's JSON: `phase2/impact_graph.json`, `phase4/consolidated.json`, `phase5/*.cc.json`, the Gradle logs. |

## Static impact tab in depth

Each row is a `FileImpact` record from the JSON contract:

| Column | Source |
|---|---|
| Path | `file.file_path` |
| Source set | `file.source_set` (`common`, `android`, `ios`, …) |
| Level | `file.relation` (`direct` / `transitive` / `expect_actual`) |
| Distance | `file.distance` (BFS hops from the nearest direct seed) |
| Matched imports | `file.imports_from_dependency` |
| Reached from | `file.propagated_from[-1]` if transitive |
| Declarations | `file.declarations` |
| `rloc` / `mcc` | `file.metrics` |

Detected `expect`/`actual` pairs are listed in a separate panel sourced from `impact_graph.expect_actual_pairs`, not on the file row.

Clicking a node in the propagation tree highlights its parents up to a direct seed.

## UI impact tab in depth

The DroidBot UI Transition Graph diff. State identity is based on the activity name plus the set of clickable element IDs — DroidBot's default heuristic. The diff lists:

- States present in AFTER but not in BEFORE (`new_screens`).
- States present in BEFORE but not in AFTER (`missing_screens`).
- Edges added or removed.

If the report shows `BLOCKED`, click the **Phase 3 log** link to see the Gradle build failure or DroidBot crash trace.

## Traceability tab

A best-effort file → screen mapping. The analyzer matches a Kotlin file's package to Compose composables declared under nearby UI packages. This is a *reviewer aid*, not a coverage proof — a file may legitimately impact a screen that the mapping cannot infer statically.

## Visualization tab

The CodeCharta JSON is rendered in-browser through the embedded viewer (when assets are present) or as a static SVG fallback. The encoding is constant across runs:

| Channel | Source | Meaning |
|---|---|---|
| Building area | `rloc` | Real lines of code per `.kt` file. |
| Building height | `mcc` | Heuristic McCabe-like complexity. |
| Building color | `impact_level` | 0 = not impacted, 1 = transitive, 2 = direct. |

You can open the same `phase5/impact.cc.json` in the public [CodeCharta viewer](https://maibornwolff.github.io/codecharta/visualization/app/index.html) for a full-screen 3D inspection.

## Raw artifacts tab

A flat list of every JSON file in `output/`. Useful when comparing two runs side by side, or feeding the data into a downstream pipeline.

## See also

- [Architecture → Phase 5 — Visualization](../architecture/phase-5-visualization.md) — how the report is generated.
- [Reference → `ConsolidatedResult`](../reference/contracts/consolidated-result.md) — the JSON behind the Summary tab.
