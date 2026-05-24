# Installation

## From source (recommended during 0.x)

```bash
git clone https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform.git
cd KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform
python -m venv .venv
source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
```

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
