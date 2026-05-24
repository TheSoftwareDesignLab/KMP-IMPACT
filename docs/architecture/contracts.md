# Data contracts

All cross-phase data is validated by Pydantic v2 models declared in [`src/kmp_impact_analyzer/contracts.py`](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/blob/main/src/kmp_impact_analyzer/contracts.py). The same models govern both the in-memory representation and the on-disk JSON files.

This page summarises the shape of each contract. For the authoritative declaration, read the source.

## `AnalysisConfig`

The CLI surface flattened into a configuration object. Built once at the top of `pipeline.run_pipeline()`.

| Field | Type | Notes |
|---|---|---|
| `repo` | `Path` | KMP project root. |
| `dependency` | `str` | Maven group or coordinate substring. |
| `before_version`, `after_version` | `str` | Source-of-truth versions. |
| `output_dir` | `Path` | Destination for `phaseN/` and `report/`. |
| `skip_dynamic` | `bool` | Skip Phase 3. |
| `keep_shadows` | `bool` | Keep `phase1/before/` and `phase1/after/` after the run. |

## `Phase1Manifest`

```json
{
  "before": { "version": "2.3.8", "modified_files": ["gradle/libs.versions.toml"] },
  "after":  { "version": "2.3.11", "modified_files": ["gradle/libs.versions.toml"] },
  "category": "library_kotlin",
  "warnings": []
}
```

## `FileImpact`

The unit of Phase 2 output.

| Field | Type | Notes |
|---|---|---|
| `path` | `str` | Relative to the project root. |
| `source_set` | `str` | `commonMain`, `androidMain`, `iosMain`, `desktopMain`, `commonTest`, … `unknown` when the file does not live under a `src/<sourceSet>Main/kotlin/` path. |
| `impact_level` | `0 \| 1 \| 2` | 2 = direct, 1 = transitive, 0 = not impacted (rarely persisted). |
| `imports_matched` | `list[str]` | Fully qualified imports that triggered the match. |
| `propagated_from` | `list[str]` | Path of the parent file in the BFS, per level. Empty for direct files. |
| `expect_actual` | `list[ExpectActualHit]` | Detected `expect`/`actual` declarations in the file. |
| `metrics` | `FileMetrics` | `rloc`, `mcc`. |

## `ExpectActualHit`

```json
{ "kind": "class", "name": "HttpEngineFactory", "role": "expect" }
```

| Field | Values |
|---|---|
| `kind` | `class`, `object`, `function`, `property` |
| `role` | `expect`, `actual` |
| `name` | Simple name of the declaration. |

The list is informational. The analyzer does not verify that an `expect` declaration has matching `actual` implementations in every required target — pairs are surfaced for human review, not as a compatibility proof.

## `ImpactGraph`

```json
{
  "files": [ /* FileImpact[] */ ],
  "counts": {
    "direct": 2,
    "transitive": 5,
    "total": 7,
    "by_source_set": { "commonMain": 4, "androidMain": 2, "iosMain": 1 },
    "expect_actual_pairs": 1
  },
  "bump": { "dependency": "io.ktor", "before": "2.3.8", "after": "2.3.11" }
}
```

The `counts.by_source_set` field is the data source for the **source-set sunburst** preview embedded in the PR comment.

## `DynamicStatus`

A small enum living on `UIDiffResult.status`:

| Value | When |
|---|---|
| `OK` | Both APKs built and DroidBot returned UTGs. |
| `BLOCKED` | Build failed or DroidBot produced no UTG. Always paired with `blocked_reason`. |
| `SKIPPED_BY_USER` | `--skip-dynamic` was passed. |

## `UIDiffResult`

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

## `ConsolidatedReport`

The artifact consumed by the HTML renderer and the PR comment composer.

```json
{
  "bump": { /* ... */ },
  "static": { "impact_graph": { /* ... */ } },
  "dynamic": { "ui_diff": { /* ... */ } },
  "traceability": [
    { "file": "...", "screens": ["PokemonDetailScreen"] }
  ],
  "risk": "LOW",
  "recommendation": "Safe to merge after CI."
}
```

The risk function is intentionally simple and lives in [`phase4_consolidate/scorer.py`](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/blob/main/src/kmp_impact_analyzer/phase4_consolidate/scorer.py). It maps `(direct_count, transitive_count, ui_diff_count, bump.category)` to a discrete `LOW`/`MEDIUM`/`HIGH` label. Changes to the heuristic must come with new fixtures under `tests/`.
