# Contributing to KMP-IMPACT

Thanks for taking the time to contribute. This document is short on purpose — read it once and you should have everything you need to send a useful change.

## Quick links

- [Open an issue](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/issues/new/choose) — bug reports and feature requests
- [Discussions](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/discussions) — design questions, ideas
- [Roadmap and known limitations](docs/troubleshooting.md)

## Development setup

```bash
git clone https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform.git
cd KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
pytest -q
```

To preview the documentation site locally:

```bash
pip install -r docs/requirements.txt
mkdocs serve
```

## Project layout

```
src/kmp_impact_analyzer/   # tool source — one sub-package per pipeline phase
tests/                     # pytest suite, fixtures included
docs/                      # MkDocs Material site (deployed via GitHub Pages)
examples/github-action/    # reference workflow + Dependabot config
gradle-init/               # Gradle init script used by the static phase
tools/                     # auxiliary scripts (ksp dump, etc.)
```

A deeper map lives in [`docs/architecture/pipeline-overview.md`](docs/architecture/pipeline-overview.md).

## Coding rules

1. **Python 3.10+**, type hints required on public functions.
2. Cross-phase data goes through the **Pydantic v2 contracts** in `contracts.py` — never invent ad-hoc dicts.
3. New behaviour ships with a test in `tests/`. The suite must stay green on Python 3.10, 3.11 and 3.12 (CI matrix).
4. No mocks in integration paths: when an APK or UTG cannot be produced, return `BLOCKED` with an explicit reason rather than fabricating data.

## Commit and PR conventions

- One logical change per commit. Conventional-commit prefixes are encouraged:
  `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`, `ci:`.
- PR titles mirror the commit style.
- Every PR must update **[CHANGELOG.md](CHANGELOG.md)** under `## [Unreleased]`.
- Reference issues with `Closes #N` / `Refs #N` in the PR body.

## Release flow

1. Bump `version` in `pyproject.toml`.
2. Move entries from `## [Unreleased]` to a new dated section in `CHANGELOG.md`.
3. Open a release PR (`release: vX.Y.Z`).
4. After merge, tag `vX.Y.Z` on `main` and push the tag — the `release.yml` workflow handles the GitHub Release.

## Reporting security issues

Please do **not** open public issues for security reports. Email `juanvalencia@habi.co` with the details and we will coordinate a fix and a coordinated disclosure window.
