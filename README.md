<h1 align="center">KMP-IMPACT</h1>
<p align="center"><em>Reviewing Dependency Updates in Kotlin Multiplatform</em></p>

<p align="center">
  <a href="https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/actions/workflows/ci.yml/badge.svg"></a>
  <a href="https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/actions/workflows/docs.yml"><img alt="Docs" src="https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/actions/workflows/docs.yml/badge.svg"></a>
  <a href="https://thesoftwaredesignlab.github.io/KMP-IMPACT/"><img alt="Docs site" src="https://img.shields.io/badge/docs-online-2da44e"></a>
  <a href="https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-MIT-blue"></a>
  <img alt="Python" src="https://img.shields.io/badge/python-3.10%2B-3776ab">
  <img alt="Kotlin" src="https://img.shields.io/badge/kotlin-1.9%2B%20%7C%202.x-7f52ff">
</p>

<p align="center">
  <a href="#capabilities">Capabilities</a> ·
  <a href="#requirements">Requirements</a> ·
  <a href="#quickstart">Quickstart</a> ·
  <a href="#pipeline">Pipeline</a> ·
  <a href="#github-action">GitHub Action</a> ·
  <a href="https://thesoftwaredesignlab.github.io/KMP-IMPACT/">Documentation</a>
</p>

---

KMP-IMPACT is a dependency-update review tool for Kotlin Multiplatform projects. It runs as a GitHub Actions workflow on every pull request that modifies `gradle/libs.versions.toml`, produces source-set-aware impact evidence and a navigable HTML report, and posts a compact summary back to the pull request.

## Capabilities

| | |
|---|---|
| Version-catalog diff | Detects the changed alias, group, and before/after versions from `gradle/libs.versions.toml`. |
| Source-set localization | Locates impacted files by source set (`commonMain`, `androidMain`, `iosMain`, `commonTest`, …). |
| Direct + transitive propagation | BFS over project-internal Kotlin imports. Each transitive file carries its `propagated_from` parent. |
| `expect` / `actual` detection | Surfaces detected `expect`/`actual` declarations touched by the change as review targets. |
| Android UI-transition diff | Optional DroidBot exploration of the BEFORE and AFTER debug APKs. |
| Reviewer-facing artifacts | HTML report with Summary, Static, Dynamic, Traceability, CodeCharta, and Raw tabs; PR comment with previews. |
| CodeCharta export | Per-file JSON with `area=rloc`, `height=mcc`, `color=impact_level`. |
| Explicit `BLOCKED` reporting | When an APK or UTG cannot be produced, the report shows the failure with a reason. No silent green builds. |

## Requirements

The analyzer targets a Kotlin Multiplatform repository that:

- Declares versions in a Gradle version catalog at `gradle/libs.versions.toml`.
- Uses **JDK 21** in CI.
- Uses **Gradle ≥ 8.7** with AGP 8.x, or **Gradle ≥ 9.0** with AGP 9.x.
- On **Kotlin 2.x** with Compose, applies the `org.jetbrains.kotlin.plugin.compose` plugin and places `jvmToolchain(…)` at the top-level of the `kotlin { … }` block.
- Exposes an Android application module that produces a Debug APK with `./gradlew :<android-module>:assembleDebug`.

Full checklist in [`docs/getting-started/requirements.md`](docs/getting-started/requirements.md).

## Quickstart

```bash
git clone https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform.git
cd KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"

kmp-impact analyze \
  --repo /path/to/your/kmp/project \
  --dependency io.ktor \
  --before-version 2.3.8 \
  --after-version 2.3.11 \
  --output-dir output \
  --skip-dynamic

open output/report/index.html
```

Full walkthrough: [`docs/getting-started/quickstart.md`](docs/getting-started/quickstart.md).

## Pipeline

```text
PR · gradle/libs.versions.toml diff
  │
  ▼
Phase 1 — Shadow build
  │  Materialises before/ and after/ shadow copies; applies the catalog edit on AFTER.
  ▼
Phase 2 — Static analysis
  │  Tree-sitter Kotlin parser; symbol graph; BFS propagation; source-set tagging; expect/actual scan.
  ▼
Phase 3 — Dynamic analysis (optional)
  │  Build BEFORE and AFTER debug APKs; run DroidBot; UTG diff (states + edges).
  ▼
Phase 4 — Consolidation
  │  Merge static and dynamic evidence; compute risk label and traceability table.
  ▼
Phase 5 — Visualization
  │  CodeCharta JSON; HTML report; PR-comment payload.
```

Each phase reads and writes a typed JSON contract under `output/phaseN/`. Contracts are documented in [`docs/reference/contracts/`](docs/reference/contracts/).

## CLI

```bash
kmp-impact analyze --repo PATH --dependency NAME --before-version A --after-version B --output-dir OUT [--skip-dynamic]
kmp-impact run-scenario --scenario-dir DIR --output-dir OUT [--skip-dynamic]
kmp-impact detect-version-changes --before A.toml --after B.toml
kmp-impact evaluate --results phase4/consolidated.json --ground-truth ground_truth.yml
```

Full reference: [`docs/reference/cli/`](docs/reference/cli/).

## GitHub Action

Drop [`examples/github-action/impact-analysis.yml`](examples/github-action/impact-analysis.yml) and [`examples/github-action/dependabot.yml`](examples/github-action/dependabot.yml) into the target repository under `.github/`, enable GitHub Pages with `Source: GitHub Actions`, and the workflow will run on every PR that touches the version catalog.

Walkthrough: [`docs/guides/github-actions.md`](docs/guides/github-actions.md).

## Output artifacts

```text
output/
├── phase1/{before,after,manifest.json}
├── phase2/{impact_graph.json, symbol_index.json}
├── phase3/{before.utg, after.utg, ui_regressions.json}
├── phase4/consolidated.json
├── phase5/{impact.cc.json, before.cc.json, after.cc.json}
└── report/{index.html, summary.json, summary.md}
```

## Development

```bash
pip install -e ".[dev]"
pytest -q
mkdocs serve            # docs preview at http://127.0.0.1:8000
```

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md) and the [`CHANGELOG.md`](CHANGELOG.md).

## License

[MIT](LICENSE) — © 2026 Esteban Castel.
