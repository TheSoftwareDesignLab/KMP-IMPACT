# `AnalysisConfig`

The CLI surface flattened into a configuration object. Built once at the top of `pipeline.run_pipeline()`.

## Fields

| Field | Type | Notes |
|---|---|---|
| `repo` | `Path` | KMP project root. |
| `dependency` | `str` | Maven group or coordinate substring. |
| `before_version` | `str` | Source-of-truth BEFORE version. |
| `after_version` | `str` | Source-of-truth AFTER version. |
| `output_dir` | `Path` | Destination for `phaseN/` and `report/`. |
| `skip_dynamic` | `bool` | Skip Phase 3. Defaults to `false`. |
| `keep_shadows` | `bool` | Keep `phase1/before/` and `phase1/after/` after the run. Defaults to `false`. |

## Example

```python
from pathlib import Path
from kmp_impact_analyzer.contracts import AnalysisConfig

config = AnalysisConfig(
    repo=Path("/path/to/kmp/project"),
    dependency="io.ktor",
    before_version="2.3.8",
    after_version="2.3.11",
    output_dir=Path("output"),
    skip_dynamic=True,
)
```

## Consumed by

- `kmp_impact_analyzer.pipeline.run_pipeline()`
- The CLI commands map their flags to this object before invoking the pipeline.
