# Phase contracts

Every phase is a pure function from one JSON contract to the next. This page documents the boundary; for the typed schemas, see [Data contracts](contracts.md).

## Phase 1 — Shadow build

**Goal.** Materialise two shadow copies of the KMP project, one with the BEFORE version applied and one with the AFTER version. The original working tree is untouched.

**Inputs.** Path to the project, target dependency, before/after versions.

**Outputs.** `phase1/before/` and `phase1/after/` (full copies) and `phase1/manifest.json` with the resolved Maven coordinates, the modified files, and the Gradle invocation log for each side.

**Edge cases.**

- If the dependency alias is not present in `libs.versions.toml`, the phase emits `BLOCKED — unknown alias` and skips the rest of the pipeline.
- If the Gradle wrapper is older than the minimum required by the project's AGP (8.7 for AGP 8.x, 9.0 for AGP 9.x) the phase still runs but marks `WARNING` in the manifest.

## Phase 2 — Static analysis

**Goal.** Identify the files affected by the bump, both *directly* (they import a symbol from the bumped library) and *transitively* (they import a directly-affected file).

**How.**

1. Parse every `.kt` under `src/` with Tree-sitter (`tree-sitter-kotlin`).
2. Build a symbol index: file → declared symbols, file → imported symbols, file → source set.
3. Resolve the bumped Maven coordinate to a set of Kotlin package roots via the mapping table (e.g. `io.ktor:*` → `io.ktor.*`).
4. Mark every file whose imports intersect those roots as `impact_level = 2` (direct).
5. BFS over the symbol graph from each direct file. Record the parent at every step in `propagated_from`. Mark each newly reached file as `impact_level = 1` (transitive).
6. Tag every impacted file with its **source set** based on its path under `src/<sourceSet>Main/kotlin/` — `commonMain`, `androidMain`, `iosMain`, `desktopMain`, `commonTest`, etc.
7. Scan the impacted files for **`expect` / `actual`** declarations and emit the detected pairs. Pairs are listed as review targets, not as compatibility proofs.
8. Compute per-file metrics: `rloc` (real lines of code) and `mcc` (a McCabe-like heuristic). These feed Phase 5.

**Outputs.** `phase2/impact_graph.json` with one entry per impacted file:

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

**Edge cases.**

- **`plugin_or_toolchain` bumps** (AGP, KSP, the Gradle wrapper itself) intentionally produce zero static impact: the change lives outside Kotlin sources, so the BFS finds nothing. This is documented as limitation **L2**.
- **Compose Multiplatform**: the mapping table currently does not resolve `org.jetbrains.compose.*` to `androidx.compose.*` (limitation **L7**). Pull requests welcome.
- **Non-convention layouts**: when a module declares Kotlin sources outside `src/<sourceSet>Main/kotlin/`, the source set is reported as `unknown` and `expect`/`actual` pairs from that file are not resolved. See [Troubleshooting](../troubleshooting.md#l3-non-convention-source-set-layout).
- **Dependency injection**: the static phase reads `import` declarations. Bindings expressed only through Koin DSL (`single { … }`), Hilt annotations, or runtime reflection are not part of the graph (limitation **L8**).

## Phase 3 — Dynamic analysis (DroidBot)

**Goal.** Compare the UI navigation graph of the BEFORE and AFTER APKs. Detect screens that exist in only one state, edges that appear or disappear, and label them as `regression`, `addition` or `no-change`.

**How.**

1. Assemble the debug APK on the BEFORE shadow copy. If this fails, mark Phase 3 `BLOCKED — APK assembly failed (BEFORE)` and return.
2. Same for AFTER.
3. Run DroidBot against each APK for a bounded number of events. Persist `utg.js`/`utg.json` into `phase3/before.utg/` and `phase3/after.utg/`.
4. Diff the two UTGs at the state level. State identity is based on the activity name plus the set of clickable element ids — DroidBot's default heuristic.

**Outputs.** `phase3/ui_regressions.json`:

```json
{
  "status": "OK",
  "before_screens": 14,
  "after_screens": 14,
  "new_screens": [],
  "missing_screens": [],
  "new_edges": [],
  "missing_edges": []
}
```

Or, on failure:

```json
{
  "status": "BLOCKED",
  "blocked_reason": "DroidBot produced no UTG artifact"
}
```

## Phase 4 — Consolidate

**Goal.** Produce the single canonical artifact `phase4/consolidated.json` that the HTML report and the PR comment consume.

**Inputs.** `phase1/manifest.json`, `phase2/impact_graph.json`, `phase3/ui_regressions.json`.

**Outputs.** `consolidated.json` with:

- `bump` — the version change being analysed.
- `static` — copy of `impact_graph.json` plus aggregate counters.
- `dynamic` — copy of `ui_regressions.json`.
- `traceability` — best-effort mapping of impacted files to UI screens by package proximity.
- `risk` — `LOW` / `MEDIUM` / `HIGH` derived from the static and dynamic counters and the bump category.
- `recommendation` — one human-readable sentence.

## Phase 5 — Visualize

**Goal.** Produce the CodeCharta JSONs and the HTML report.

**Outputs.**

- `phase5/impact.cc.json` — per-bump delta, suitable for the CodeCharta delta view.
- `phase5/before.cc.json`, `phase5/after.cc.json` — full snapshots for side-by-side review.
- `report/index.html` — the navigable report, served from a single static directory.

The CodeCharta encoding used by KMP-IMPACT is:

| Channel | Source | Meaning |
|---|---|---|
| Building area | `rloc` | Real lines of code per `.kt` file. |
| Building height | `mcc` | Heuristic McCabe-like complexity. |
| Building color | `impact_level` | 0 = not impacted, 1 = transitive, 2 = direct. |
