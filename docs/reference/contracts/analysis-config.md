# `AnalysisConfig`

The CLI surface flattened into a configuration dataclass. Built once at the top of `pipeline.run_pipeline()`. Defined in [`src/kmp_impact_analyzer/config.py`](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/blob/main/src/kmp_impact_analyzer/config.py).

## Fields

| Field | Type | Default | Notes |
|---|---|---|---|
| `repo_path` | `str` | `""` | Path or URL to the KMP project repository. |
| `dependency_group` | `str` | `""` | Maven group or coordinate substring (e.g. `io.ktor`). |
| `before_version` | `str` | `""` | Version before the change. |
| `after_version` | `str` | `""` | Version after the change. |
| `output_dir` | `str` | `"output"` | Where shadow copies, phase outputs, and the report are written. |
| `skip_dynamic` | `bool` | `False` | Skip Phase 3 entirely. |
| `init_script_path` | `str` | `""` | Path to the Gradle init script used by the static phase. Empty means use the bundled `gradle-init/impact-analyzer-init.gradle.kts`. |
| `droidbot_timeout` | `int` | `120` | DroidBot per-event timeout in seconds. |
| `droidbot_policy` | `str` | `"dfs_greedy"` | DroidBot exploration policy. |
| `before_apk` | `str` | `""` | Optional pre-built APK to reuse for the BEFORE state. |
| `after_apk` | `str` | `""` | Optional pre-built APK to reuse for the AFTER state. |
| `droidbot_before_output` | `str` | `""` | Optional pre-generated DroidBot output directory for the BEFORE APK. |
| `droidbot_after_output` | `str` | `""` | Optional pre-generated DroidBot output directory for the AFTER APK. |
| `extra_seed_packages` | `list[str]` | `[]` | Additional Kotlin package roots to treat as impact seeds, on top of the auto-resolved ones. |

## Construction helpers

```python
from pathlib import Path
from kmp_impact_analyzer.config import AnalysisConfig

# From CLI keyword arguments (extra keys are ignored)
config = AnalysisConfig.from_cli(
    repo_path="/path/to/kmp/project",
    dependency_group="io.ktor",
    before_version="2.3.8",
    after_version="2.3.11",
    output_dir="output",
    skip_dynamic=True,
)

# From a YAML file (scenario)
config = AnalysisConfig.from_yaml(Path("scenarios/ktor/scenario.yml"))
```

`from_yaml` reads every key from the YAML file and silently discards keys that are not fields of the dataclass. This is what makes `scenario.yml` forward-compatible with new fields.

## Consumed by

- `kmp_impact_analyzer.pipeline.run_pipeline()`
- The four CLI subcommands map their flags to this object before invoking the pipeline.

## See also

- [`analyze`](../cli/analyze.md) — the most common way to build this config.
- [`run-scenario`](../cli/run-scenario.md) — builds it from `scenario.yml`.
