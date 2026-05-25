# Changelog

All notable changes to this project are documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Project documentation site (MkDocs Material) with sections for getting-started, usage, architecture, presets, evaluation, and troubleshooting.
- Dependabot configuration for the tool itself (`pip`, `github-actions`).
- CI workflow running `pytest` against Python 3.10, 3.11 and 3.12.
- Example GitHub Action and Dependabot configuration ready to copy into a target KMP repo.

## [0.1.0] — 2026-05-24

First public release of **KMP-IMPACT**: the thesis-grade dependency impact analyzer for Kotlin Multiplatform projects.

### Added
- 5-phase pipeline: shadow build, static analysis (Tree-sitter + BFS), dynamic analysis (DroidBot UTG diff), consolidation, visualization.
- CLI commands `analyze`, `run-scenario`, `detect-version-changes`, `evaluate`.
- HTML report with Summary, Static Impact, UI Impact, Traceability, Visualization and Raw Artifacts tabs.
- CodeCharta exporter (`area=rloc`, `height=mcc`, `color=impact_level`).
- Propagation tree visualization with per-level expand/collapse and persisted `propagated_from` parent links.
- Pydantic v2 contracts for every inter-phase JSON artifact.
- Reference integration for KMP demo projects: Pokedex, Confetti, DroidconKotlin, KMP-App-Template, kmp-production-sample.

### Known limitations
- L2 — `plugin_or_toolchain` bumps report 0 static impact (by design).
- L4 — Dynamic phase requires a building APK and a running emulator.
- L6 — Concurrent `actions/deploy-pages@v4` runs across multiple repos under the same user may cancel each other.
- L7 — Internal package-to-Maven mapping does not yet resolve `org.jetbrains.compose.*` ⇄ `androidx.compose.*` (Compose Multiplatform), which lowers F1 for repos that depend on it.

[Unreleased]: https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/releases/tag/v0.1.0
