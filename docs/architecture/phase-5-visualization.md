# Phase 5 — Visualization

The fifth phase produces the CodeCharta JSON exports and the HTML report.

## Inputs

- `phase4/consolidated.json` — the canonical artifact.
- All previous phase artifacts (linked from the *Raw Artifacts* tab in the report).

## Outputs

```text
phase5/
├── impact.cc.json          # per-bump delta, suitable for CodeCharta delta view
├── before.cc.json          # full snapshot, BEFORE state
└── after.cc.json           # full snapshot, AFTER state

report/
├── index.html              # the navigable report
├── summary.json
└── summary.md              # PR-comment payload
```

## CodeCharta encoding

| Channel | Source | Meaning |
|---|---|---|
| Building area | `rloc` | Real lines of code per `.kt` file. |
| Building height | `mcc` | Heuristic McCabe-like complexity. |
| Building color | `impact_level` | 0 = not impacted, 1 = transitive, 2 = direct. |

Open `impact.cc.json` in the [CodeCharta viewer](https://maibornwolff.github.io/codecharta/visualization/app/index.html) for the full-screen 3D view. The embedded viewer in the HTML report uses the same JSON.

## HTML report structure

The report is a self-contained directory. It can be served from any static file host. Six tabs:

| Tab | Content source |
|---|---|
| Summary | `consolidated.summary` + sunburst from `counts.by_source_set` |
| Static impact | `consolidated.static.impact_graph.files` + propagation tree |
| UI impact | `consolidated.dynamic.ui_diff` + embedded UTG iframe |
| Traceability | `consolidated.traceability` |
| Visualization | `phase5/impact.cc.json` rendered through CodeCharta |
| Raw artifacts | Filesystem listing of all `phaseN/` JSON files |

Full tab-by-tab walkthrough in [Guides → Reading the Report](../guides/reading-the-report.md).

## PR-comment payload

The `summary.md` file is the payload posted as a PR comment by the GitHub Action. It includes the bump, the headline counters, the source-set distribution, the `expect`/`actual` count, the risk label, the recommendation, and the URL of the published HTML report.

## Edge cases

- **No CodeCharta viewer assets.** The Visualization tab falls back to a static SVG. See [L5](../troubleshooting.md#l5-codecharta-viewer-requires-embedded-assets).
- **Pages cancellation.** The HTML is still produced and uploaded as a workflow artifact even when `deploy-pages` is cancelled. See [L6](../troubleshooting.md#l6-deploy-pages-cancellation).

## Contracts

The structures rendered into the HTML are documented in [Reference → Data Contracts](../reference/contracts/index.md). The renderer itself lives in `src/kmp_impact_analyzer/reporting/`.
