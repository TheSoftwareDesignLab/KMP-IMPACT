# Data Contracts

The Pydantic v2 models that travel between phases. The same models govern both the in-memory representation and the on-disk JSON files. The authoritative declaration lives in [`src/kmp_impact_analyzer/contracts.py`](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/blob/main/src/kmp_impact_analyzer/contracts.py).

## Contracts

| Type | Purpose |
|---|---|
| [`AnalysisConfig`](analysis-config.md) | The CLI surface flattened into a configuration object. |
| [`FileImpact`](file-impact.md) | The unit of Phase 2 output — one impacted Kotlin file. |
| [`ExpectActualHit`](expect-actual-hit.md) | A detected `expect`/`actual` declaration. |
| [`ImpactGraph`](impact-graph.md) | The aggregate Phase 2 output. |
| [`UIDiffResult`](ui-diff-result.md) | The Phase 3 output — UI Transition Graph diff. |
| [`ConsolidatedReport`](consolidated-report.md) | The Phase 4 canonical artifact consumed by the HTML report and the PR comment. |

## Conventions

- All contracts are versioned by the package. A schema change is a minor-version bump (`0.X.0`) until 1.0.
- Optional fields default to `null` or to the empty collection. Consumers must tolerate missing optional fields when reading older artifacts.
- `BLOCKED` is a first-class status, not an exception. Every phase whose output can fail carries a `status` field with `OK`, `BLOCKED`, or `SKIPPED_BY_USER`.

## Working with the JSON on disk

The on-disk JSON files exactly mirror the Pydantic model dumps (`model_dump(by_alias=True)`). To consume them from another language, generate types from the JSON Schema:

```bash
python -c "
from kmp_impact_analyzer.contracts import ConsolidatedReport
import json; print(json.dumps(ConsolidatedReport.model_json_schema(), indent=2))
" > consolidated_report.schema.json
```
