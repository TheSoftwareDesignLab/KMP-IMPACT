# Phase 1 — Shadow build

The first phase materialises two shadow copies of the KMP project, one with the BEFORE version applied and one with the AFTER version. The original working tree is never touched.

## Inputs

- Path to the project root.
- Target dependency (Maven group or coordinate substring).
- `before_version`, `after_version`.

## Outputs

```text
phase1/
├── before/         # full project copy, BEFORE state
├── after/          # full project copy, AFTER state
└── manifest.json   # resolved coordinates, modified files, Gradle log per side
```

The contract written here is consumed by Phase 2 (static analysis) and Phase 3 (dynamic analysis).

## What happens

1. Resolve the dependency alias in `gradle/libs.versions.toml`. Bail out with `BLOCKED — unknown alias` if not found.
2. Copy the entire project tree to `phase1/before/`. The copy honours `.gitignore` to skip caches and build outputs.
3. Copy the project tree again to `phase1/after/`.
4. Edit `phase1/after/gradle/libs.versions.toml` so the alias points to `after_version`.
5. Run the Gradle wrapper version check on each shadow copy and capture the log into `manifest.json`.

## Edge cases

- **Unknown alias.** The alias does not exist in the version catalog. Emits `BLOCKED — unknown alias` and skips the rest of the pipeline.
- **Outdated wrapper.** The Gradle wrapper is older than the minimum required by the project's AGP (8.7 for AGP 8.x, 9.0 for AGP 9.x). The phase still runs but adds a `WARNING` entry to `manifest.json`. Phase 3 may upgrade the wrapper in place for the duration of the build.
- **Catalog parse error.** The TOML file cannot be parsed. The phase reports the line number and exits.

## Contract

See [Reference → `Phase1Manifest`](../reference/contracts/index.md). The relevant fields:

```json
{
  "before": { "version": "2.3.8",  "modified_files": ["gradle/libs.versions.toml"] },
  "after":  { "version": "2.3.11", "modified_files": ["gradle/libs.versions.toml"] },
  "category": "library_kotlin",
  "warnings": []
}
```

The `category` field drives the downstream behaviour. Phase 2 reports zero static impact for `plugin_or_toolchain` bumps by design — see [L2](../troubleshooting.md#l2-plugin_or_toolchain-bumps-report-zero-static-impact).

## Next phase

Phase 2 reads `phase1/before/` and `phase1/after/` and parses every `.kt` file to build the symbol graph.

→ [Phase 2 — Static analysis](phase-2-static-analysis.md)
