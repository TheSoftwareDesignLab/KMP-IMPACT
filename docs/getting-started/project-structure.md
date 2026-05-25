# Project Structure

A short tour of the repository so you know where to look.

```text
KMP-IMPACT/
├── src/
│   └── kmp_impact_analyzer/
│       ├── cli.py                  # Click entry points
│       ├── config.py               # AnalysisConfig
│       ├── contracts.py            # Pydantic v2 cross-phase models
│       ├── github_version_change.py
│       ├── pipeline.py             # run_pipeline()
│       ├── phase1_shadow/          # before/ + after/ shadow copies
│       ├── phase2_static/          # parser, symbol graph, BFS, source-set tagging
│       ├── phase3_dynamic/         # DroidBot wrapper + UTG diff
│       ├── phase4_consolidate/     # merge + risk label
│       ├── phase5_visualize/       # CodeCharta exporters
│       ├── reporting/              # HTML report generator
│       ├── compat/                 # stack compatibility helpers
│       ├── evaluation/             # ground-truth comparison
│       └── utils/                  # logging, JSON helpers
├── tests/                          # pytest suite + fixtures
├── tools/                          # auxiliary scripts (ksp dump, …)
├── gradle-init/                    # Gradle init script used by Phase 2
├── docs/                           # this MkDocs Material site
├── examples/
│   └── github-action/              # drop-in workflow + Dependabot config
├── pyproject.toml                  # package metadata + dev dependencies
├── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md
└── LICENSE
```

## Where each phase lives

| Phase | Module |
|---|---|
| 1 — Shadow build | `src/kmp_impact_analyzer/phase1_shadow/` |
| 2 — Static analysis | `src/kmp_impact_analyzer/phase2_static/` |
| 3 — Dynamic analysis | `src/kmp_impact_analyzer/phase3_dynamic/` |
| 4 — Consolidation | `src/kmp_impact_analyzer/phase4_consolidate/` |
| 5 — Visualization | `src/kmp_impact_analyzer/phase5_visualize/` |

The data contracts that travel between phases live in [`src/kmp_impact_analyzer/contracts.py`](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/blob/main/src/kmp_impact_analyzer/contracts.py). See [Reference → Data Contracts](../reference/contracts/index.md) for the field-by-field documentation.

## Output layout

A successful run produces:

```text
output/
├── phase1/{before,after,manifest.json}
├── phase2/{impact_graph.json, symbol_index.json}
├── phase3/{before.utg, after.utg, ui_regressions.json}
├── phase4/consolidated.json
├── phase5/{impact.cc.json, before.cc.json, after.cc.json}
└── report/{index.html, summary.json, summary.md}
```

Every phase reads the previous phase's JSON and emits its own under `output/phaseN/`. You can re-run a single phase by pointing the CLI at an existing `output/` directory.
