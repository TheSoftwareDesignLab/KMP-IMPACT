# Data Contracts

The Pydantic v2 models that travel between phases, plus the `AnalysisConfig` dataclass. The same models govern both the in-memory representation and the on-disk JSON files. The authoritative declaration lives in [`src/kmp_impact_analyzer/contracts.py`](https://github.com/EstebanCastel/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/blob/main/src/kmp_impact_analyzer/contracts.py).

## Contracts

| Type | Purpose | Page |
|---|---|---|
| `AnalysisConfig` | The CLI surface flattened into a configuration dataclass. | [Open](analysis-config.md) |
| `VersionChange` | One detected catalog change — emitted by `detect-version-changes`. | [Open](#versionchange) |
| `ShadowManifest` | Phase 1 output: where the BEFORE and AFTER trees live on disk. | [Open](#shadowmanifest) |
| `FileImpact` | One impacted Kotlin file (Phase 2). | [Open](file-impact.md) |
| `ExpectActualPair` | A detected `expect`/`actual` pairing (Phase 2). | [Open](expect-actual-pair.md) |
| `ImpactGraph` | Phase 2 aggregate output. | [Open](impact-graph.md) |
| `UIRegressions` | Phase 3 dynamic output — UTG diff result. | [Open](ui-regressions.md) |
| `ConsolidatedResult` | Phase 4 canonical artefact consumed by the HTML report and the PR comment. | [Open](consolidated-result.md) |
| `EvaluationResult` | `kmp-impact evaluate` output — Precision / Recall / F1 plus TP/FP/FN lists. | [Open](#evaluationresult) |

## Conventions

- All Pydantic contracts use `from_attributes=False`. Fields default to `null` or to the empty collection.
- Enum values are lowercase strings on disk (`"direct"`, `"transitive"`, `"completed"`, `"blocked"`, `"skipped"`).
- `AnalysisConfig` is a plain `@dataclass`, not a Pydantic model — it is constructed from YAML or CLI flags and is never serialised to disk in the canonical pipeline.
- The HTML renderer computes some derived values (risk label, recommendation prose) that are **not** persisted in `ConsolidatedResult`. They are generated at report time.

## Working with the JSON on disk

To consume the artefacts from another language, generate the JSON Schema:

```bash
python -c "
from kmp_impact_analyzer.contracts import ConsolidatedResult
import json; print(json.dumps(ConsolidatedResult.model_json_schema(), indent=2))
" > consolidated_result.schema.json
```

## Smaller helper types

A few short types are documented inline rather than on their own pages.

### `VersionChange` { #versionchange }

```json
{
  "dependency_group": "io.ktor",
  "version_key": "ktor",
  "before": "2.3.8",
  "after": "2.3.11"
}
```

Returned in batches inside the `VersionChangeSet` emitted by `kmp-impact detect-version-changes`.

### `ShadowManifest` { #shadowmanifest }

```json
{
  "before_dir": "output/phase1/before",
  "after_dir":  "output/phase1/after",
  "version_change": { /* VersionChange */ },
  "init_script_injected": true,
  "detekt_reports": {},
  "kover_reports": {},
  "compilation_after": {},
  "compile_targets": { ":android": true, ":shared": true }
}
```

`compile_targets` records the result of `./gradlew :<module>:assembleDebug` for each detected Android target. Failed targets are still listed with `false`, which is how Phase 3 decides whether to mark itself `BLOCKED`.

### `EvaluationResult` { #evaluationresult }

```json
{
  "scenario": "pokedex_ktor_minor",
  "precision": 0.83, "recall": 0.75, "f1": 0.79,
  "true_positives":  ["..."],
  "false_positives": ["..."],
  "false_negatives": ["..."],
  "screen_precision": 1.0, "screen_recall": 1.0, "screen_f1": 1.0,
  "screen_tp": [], "screen_fp": [], "screen_fn": []
}
```

Written by [`kmp-impact evaluate`](../cli/evaluate.md).
