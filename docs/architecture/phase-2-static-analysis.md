# Phase 2 — Static analysis

The second phase identifies the Kotlin files affected by the bump — both *directly* (they import a symbol from the bumped library) and *transitively* (they import a directly-affected file) — and tags every impacted file with its source set.

## Inputs

- `phase1/before/`, `phase1/after/` — the shadow copies from Phase 1.
- `phase1/manifest.json` — the resolved coordinates and bump category.

## Outputs

```text
phase2/
├── impact_graph.json
└── symbol_index.json
```

## What happens

1. Parse every `.kt` under `src/` with Tree-sitter (`tree-sitter-kotlin`).
2. Build a symbol index: file → declared symbols, file → imported symbols, file → source set.
3. Resolve the bumped Maven coordinate to a set of Kotlin package roots via the [`MAVEN_TO_KOTLIN`](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/blob/main/src/kmp_impact_analyzer/phase2_static/dependency_mapping.py) mapping table (e.g. `io.ktor:*` → `io.ktor.*`).
4. Mark every file whose imports intersect those roots as `impact_level = 2` (direct).
5. BFS over the symbol graph from each direct file. Record the parent at every step in `propagated_from`. Mark each newly reached file as `impact_level = 1` (transitive).
6. Tag every impacted file with its **source set** based on its path under `src/<sourceSet>Main/kotlin/`.
7. Scan the impacted files for `expect` / `actual` declarations. Pairs are surfaced as review targets — not compatibility proofs.
8. Compute per-file metrics: `rloc` (real lines of code), `mcc` (a McCabe-like heuristic). These feed Phase 5.

## Output sample

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

Aggregate counters per source set are written under `impact_graph.json#counts.by_source_set` and rendered as the sunburst preview in the PR comment.

## Edge cases

- **`plugin_or_toolchain` bumps** — AGP, KSP, the Gradle wrapper. Zero static impact by design (no Kotlin imports). See [L2](../troubleshooting.md#l2-plugin_or_toolchain-bumps-report-zero-static-impact).
- **Compose Multiplatform** — `org.jetbrains.compose.*` coordinates are not yet mapped to `androidx.compose.*` package roots. See [L7](../troubleshooting.md#l7-compose-multiplatform-mapping).
- **Non-convention source-set layout** — files outside `src/<sourceSet>Main/kotlin/` are tagged `unknown`. See [L3](../troubleshooting.md#l3-non-convention-source-set-layout).
- **Dependency injection** — bindings expressed only through Koin DSL, Hilt annotations or reflection are not in the graph. See [L8](../troubleshooting.md#l8-dependency-injection-misses).

## Contracts

- [`FileImpact`](../reference/contracts/file-impact.md)
- [`ExpectActualHit`](../reference/contracts/expect-actual-hit.md)
- [`ImpactGraph`](../reference/contracts/impact-graph.md)

## Next phase

Phase 3 reads the same shadow copies and builds the APKs for the dynamic analysis. It does **not** read Phase 2 output — the two phases run in parallel in CI.

→ [Phase 3 — Dynamic analysis](phase-3-dynamic-analysis.md)
