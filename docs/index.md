---
hide:
  - navigation
  - toc
---

<div class="kmp-hero">
  <h1>KMP-IMPACT</h1>
  <p class="tagline">Reviewing Dependency Updates in Kotlin Multiplatform.</p>
  <p>
    <a class="md-button md-button--primary" href="getting-started/quickstart/">Quickstart</a>
    <a class="md-button" href="usage/github-action/">GitHub Action</a>
    <a class="md-button" href="architecture/overview/">Architecture</a>
  </p>
</div>

KMP-IMPACT runs as a GitHub Actions workflow on every pull request that modifies `gradle/libs.versions.toml`. For each version-catalog change it produces source-set-aware impact evidence (direct and transitive Kotlin files, source-set distribution, detected `expect`/`actual` pairs) and, when an Android emulator is available, a UI-transition graph diff between the BEFORE and AFTER APKs. The result is a navigable HTML report on GitHub Pages and a compact comment back on the pull request.

<div class="kmp-grid" markdown>

<div class="card" markdown>
### :material-source-branch: Source-set aware
Locates impacted files in `commonMain`, `androidMain`, `iosMain`, `commonTest`, … Reports detected `expect`/`actual` pairs as review targets — not compatibility proofs.
</div>

<div class="card" markdown>
### :material-graph: Static propagation
Tree-sitter Kotlin parser; symbol graph; BFS from direct impact seeds. Each transitive file carries its `propagated_from` parent.
</div>

<div class="card" markdown>
### :material-cellphone-android: DroidBot UI diff
Builds BEFORE and AFTER debug APKs, launches DroidBot on an Android emulator, diffs the resulting UI Transition Graphs.
</div>

<div class="card" markdown>
### :material-cube-outline: CodeCharta
Per-file JSON with `area=rloc`, `height=mcc`, `color=impact_level`. Drop into [CodeCharta](https://maibornwolff.github.io/codecharta/visualization/app/index.html) for the 3D inspection.
</div>

<div class="card" markdown>
### :material-alert-circle-outline: Explicit BLOCKED
When the APK or UTG cannot be produced, the report shows `BLOCKED` with the failure reason. No silent green builds. No fabricated dynamic evidence.
</div>

<div class="card" markdown>
### :material-source-pull: PR-native
GitHub Actions + Dependabot integration. PR comment as the triage entry point, full HTML report on GitHub Pages for drill-down.
</div>

</div>

## Evaluation headline

Across 53 real Dependabot pull requests in 5 public KMP projects, against a manually audited ground truth (121/121 dependencies cross-checked, 89/89 direct files verified file-by-file).

| Repo | PRs | Pipeline OK | F1 (static) |
|---|---:|---:|---:|
| Pokedex | 10 | 8 | 0.699 |
| Confetti | 11 | 8 | 0.737 |
| DroidconKotlin | 11 | 9 | 0.828 |
| KMP-App-Template | 10 | 9 | 0.307 |
| kmp-production-sample | 11 | 10 | 0.780 |
| **Total** | **53** | **44 (95.7 % adjusted)** | **F1 = 0.658 · Recall = 0.720** |

Methodology, per-PR table, and audit details: [Evaluation](evaluation/index.md).

## Where to start

| If you are… | Open… |
|---|---|
| Installing the analyzer for the first time | [Getting started → Requirements](getting-started/requirements.md) |
| Looking for the CLI surface | [Usage → CLI reference](usage/cli.md) |
| Wiring the analyzer into your CI | [Usage → GitHub Action](usage/github-action.md) |
| Reading the internals or writing a contribution | [Architecture → Pipeline overview](architecture/overview.md) |
| Migrating a specific KMP project | [Presets](presets/index.md) |
| Diagnosing a `BLOCKED` phase | [Troubleshooting](troubleshooting.md) |
