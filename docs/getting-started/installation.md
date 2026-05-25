# Installation

## From source (recommended during 0.x)

```bash
git clone https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform.git
cd KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform
python -m venv .venv
source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
```

For a line-by-line account of what each command downloads or creates, see [What gets installed](what-gets-installed.md).

The package exposes a single console entry point:

```bash
kmp-impact --help
```

## Smoke test

```bash
pytest -q
```

You should see all unit tests pass. The integration tests in `tests/` use Kotlin fixtures shipped with the repository, so no external project is required.

## Optional: documentation site

If you plan to contribute to the docs:

```bash
pip install -r docs/requirements.txt
mkdocs serve
# open http://127.0.0.1:8000/
```

## Verify the CLI

```bash
kmp-impact --version
kmp-impact analyze --help
kmp-impact detect-version-changes --help
```

If those commands respond, the installation is complete — head to the [Quickstart](quickstart.md).

## Next steps

- [Quickstart](quickstart.md) — run a first analysis on a sample KMP project.
- [Requirements](requirements.md) — make sure your target KMP project meets the prerequisites.
- [What gets installed](what-gets-installed.md) — understand the disk, network, and dependency footprint.
- [Guides → Preparing an existing KMP project](../guides/preparing-a-kmp-project.md) — checklist to adapt a real KMP repo before wiring CI.
- [Guides → How everything talks to each other](../guides/how-it-works.md) — see the runtime flow before wiring the workflow into a real repo.
