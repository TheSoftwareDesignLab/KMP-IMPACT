---
hide:
  - navigation
---

# Introduction

KMP-IMPACT is a PR-native dependency-impact analyzer for **Kotlin Multiplatform** projects. It runs as a GitHub Actions workflow on every pull request that modifies `gradle/libs.versions.toml` and turns the version-catalog diff into source-set-aware review evidence: direct and transitive Kotlin files, detected `expect`/`actual` pairs, optional Android UI-transition diffs, and a navigable HTML report linked from the PR comment.

<div class="kmp-callout" markdown>
**New here?** Start with [Installation](getting-started/installation.md), then run a first analysis from the [Quickstart](getting-started/quickstart.md). Wiring it into CI? Jump to [Configuring GitHub Actions](guides/github-actions.md).
</div>

## Documentation map

<div class="kmp-grid" markdown>

<div class="card" markdown>
### Getting Started
Install the analyzer, learn the prerequisites, and run a first analysis.

[Open →](getting-started/index.md)
</div>

<div class="card" markdown>
### Guides
Task-oriented walkthroughs: analyzing locally, wiring GitHub Actions, biasing Dependabot, reading the report, diagnosing failures.

[Open →](guides/index.md)
</div>

<div class="card" markdown>
### Architecture
The 5-phase pipeline, one page per phase, plus the data-flow overview.

[Open →](architecture/index.md)
</div>

<div class="card" markdown>
### API Reference
Every CLI command, every data contract, and the GitHub Action inputs and outputs.

[Open →](reference/index.md)
</div>

<div class="card" markdown>
### Presets
Drop-in integrations for Pokedex, Confetti, DroidconKotlin, KMP-App-Template, and kmp-production-sample.

[Open →](presets/index.md)
</div>

<div class="card" markdown>
### Troubleshooting
Symptom → cause → fix for every documented limitation L1 through L8.

[Open →](troubleshooting.md)
</div>

</div>

## At a glance

- **Single workflow.** One `impact-analysis.yml` file in `.github/workflows/`. No external services to run.
- **Source-set aware.** Reports impacted files by `commonMain`, `androidMain`, `iosMain`, `desktopMain`, `commonTest`.
- **`expect`/`actual` detection.** Surfaces detected pairs as review targets — not compatibility proofs.
- **Optional Android UI diff.** DroidBot UTGs of the BEFORE and AFTER APKs when an emulator is available.
- **CodeCharta export.** Per-file JSON with `area=rloc`, `height=mcc`, `color=impact_level`.
- **Explicit `BLOCKED`.** No silent green builds; failed phases carry a `blocked_reason`.

## Where the code lives

| Concern | Path |
|---|---|
| CLI entry point | `src/kmp_impact_analyzer/cli.py` |
| Orchestrator | `src/kmp_impact_analyzer/pipeline.py` |
| Cross-phase models | `src/kmp_impact_analyzer/contracts.py` |
| Phase modules | `src/kmp_impact_analyzer/phase{1..5}_*/` |
| Reporting | `src/kmp_impact_analyzer/reporting/` |
| Tests | `tests/` |
| Docs site | `docs/`, `mkdocs.yml` |
| Example GitHub Action | `examples/github-action/` |
