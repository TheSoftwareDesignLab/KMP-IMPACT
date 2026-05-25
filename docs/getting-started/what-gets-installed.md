# What gets installed

A line-by-line walkthrough of what each install step downloads, creates, or modifies — on your machine, on your KMP repository, and inside the GitHub Actions runner when the workflow runs.

<div class="kmp-callout" markdown>
**Mental model.** KMP-IMPACT installs in **two places**: a Python package on the machine that runs the analyzer (locally or in CI), and three files inside the target KMP repository (the workflow, the Dependabot config, and the vendored analyzer source). It does **not** modify your application code, your `main` branch, or any of your existing build configuration.
</div>

## On your local machine

Walking through the quickstart, step by step:

### `git clone https://github.com/EstebanCastel/KMP-IMPACT-…`

| Effect | Detail |
|---|---|
| Creates | A directory `KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/` in the current working directory. |
| Downloads | The full history of the repo — roughly **3 MB** including code, tests, docs, and example workflow. |
| Modifies | Nothing outside the cloned directory. |

### `cd KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform`

Changes directory. No filesystem writes.

### `python -m venv .venv`

| Effect | Detail |
|---|---|
| Creates | A `.venv/` directory at the repo root. |
| Downloads | Nothing — `venv` is part of the standard library. |
| Size | ~30 MB (a bare Python interpreter copy plus `pip`). |
| Modifies | Nothing outside `.venv/`. |

### `source .venv/bin/activate`

Adjusts your shell's `PATH` so `python` and `pip` resolve to the venv's copy for the rest of the session. No filesystem writes.

### `pip install -e ".[dev]"`

This is the only step that pulls external dependencies. The `-e` flag installs the analyzer in editable mode, so subsequent edits to `src/` are picked up without re-installing.

**Runtime dependencies** (declared in `pyproject.toml`):

| Package | Purpose |
|---|---|
| `click >= 8.1` | CLI argument parsing — defines the `kmp-impact` subcommands. |
| `pydantic >= 2.0` | Validates every cross-phase JSON contract. |
| `pyyaml >= 6.0` | Reads `scenario.yml` and `ground_truth.yml`. |
| `tree-sitter >= 0.21, < 0.24` | Generic Tree-sitter runtime used by the static phase. |
| `tree-sitter-kotlin >= 0.1` | Kotlin grammar binding for Tree-sitter. |
| `rich >= 13.0` | Pretty console output for the CLI. |

**Dev dependencies** (only with `[dev]`):

| Package | Purpose |
|---|---|
| `pytest >= 7.0` | Test runner. |
| `pytest-cov >= 4.0` | Coverage report for the test suite. |

The full transitive closure with `[dev]` is around **15–20 MB** installed in `.venv/lib/python3.X/site-packages/`. Pip downloads wheels from PyPI; there are no native compile steps on macOS, Linux, or Windows for any of these packages.

### `kmp-impact analyze --repo /path/to/your/kmp/project …`

Now the analyzer runs against a real KMP project. Even though the analyzer lives inside `.venv/`, the **target project on disk is what produces the side effects** — your KMP repo on disk is the work surface for the pipeline.

| Effect | Detail |
|---|---|
| Reads | `gradle/libs.versions.toml` from `--repo`. Then every `.kt` file under `src/`. |
| Writes | Two shadow copies of the project under `<output-dir>/phase1/before/` and `<output-dir>/phase1/after/`. JSON artifacts under `<output-dir>/phase{2..5}/`. The HTML report under `<output-dir>/report/`. |
| Modifies | The AFTER shadow copy's `gradle/libs.versions.toml` only — never the original project's files. |
| Downloads | Nothing additional from the static phase. If you drop `--skip-dynamic`, Gradle downloads the project's Maven dependencies and the Android Gradle Plugin assembles two debug APKs. That can be a few hundred MB on first run; subsequent runs reuse Gradle's cache. |
| Side effects | If you drop `--skip-dynamic` and `--*-apk` / `--droidbot-*-output` are not provided, the analyzer launches DroidBot against an Android emulator. The emulator and DroidBot must already be available on your machine. |

Rough disk-usage budget for a single full run on a medium KMP project (~50–100 Kotlin files):

| Item | Size |
|---|---|
| `phase1/before/` + `phase1/after/` | 50–400 MB (≈ project size × 2) |
| `phase2/`, `phase3/`, `phase4/` JSON | 1–5 MB |
| `phase5/*.cc.json` | 1–3 MB |
| `report/` | 1–2 MB (no large media, just HTML + small SVG) |
| Gradle build caches (if dynamic) | 200 MB – 1 GB on first run, reused thereafter |

Use `--keep-shadows` if you want to inspect or re-run a single phase; otherwise the shadow copies are deleted at the end of the run.

### `open output/report/index.html`

Opens the HTML report in your default browser. No filesystem writes.

## On your target KMP repository (CI install)

When you wire KMP-IMPACT into a KMP project — following [Configuring GitHub Actions](../guides/github-actions.md) — three things land in the target repo:

| Path | Size | What it does |
|---|---|---|
| `.github/workflows/impact-analysis.yml` | ~50 KB | The five-job pipeline workflow. |
| `.github/dependabot.yml` | ~2 KB | Tells Dependabot what to scan and which majors to skip. |
| `tools/kmp-impact-analyzer/` | ~3 MB | Vendored copy of this repository. |

That's it. **Nothing inside `src/`, `gradle/`, `build.gradle.kts`, or your app modules is modified.** The workflow only *reads* your `gradle/libs.versions.toml`; it never edits the version catalog on `main`.

Optional: enable GitHub Pages with `Source: GitHub Actions`. That toggles a setting on the repository but does not create any file on its own. The workflow creates a `gh-pages-history` branch on the first deploy, which carries the cumulative report directory.

## Inside the GitHub Actions runner

When the workflow runs, the runner downloads more transitively but none of it persists in your repo:

| Phase | What the runner installs / downloads |
|---|---|
| `detect` | Python 3.11, the analyzer's deps (`click`, `pydantic`, `tree-sitter`, …) — ~15 MB. |
| `static-pipeline` | Same as `detect`, then runs the Gradle wrapper for module discovery — depends on your project, typically 200–500 MB the first time and cached after. |
| `droidbot` | The Android SDK (~3 GB) plus the system image, DroidBot itself, `androguard`. Heaviest job by far. Cached aggressively across runs. |
| `merge` | Re-installs the analyzer, plus `matplotlib` for the sunburst PNG embedded in the PR comment. |
| `deploy-pages` | `actions/upload-pages-artifact` and `actions/deploy-pages` only. No additional install. |

All caches live inside the runner and are reused across PRs via `actions/cache@v4`. The repo itself never grows because of these downloads.

## What KMP-IMPACT does **not** install

To make the boundary explicit:

- It does **not** modify your `build.gradle.kts`, `settings.gradle.kts`, `gradle.properties`, or any source file under `src/`.
- It does **not** push to your `main` branch. The only branch the workflow writes to is `gh-pages-history`, which exists solely to serve the cumulative report directory.
- It does **not** read or require any secret. The workflow uses only the auto-provisioned `${{ secrets.GITHUB_TOKEN }}`.
- It does **not** invoke external services beyond Maven Central, Google Maven, the Gradle plugin portal, PyPI, and GitHub itself. Reports are not sent off-platform.

## Removing KMP-IMPACT

If you decide to remove the analyzer from a target repo:

```bash
rm -rf .github/workflows/impact-analysis.yml
rm -rf .github/dependabot.yml      # only if Dependabot is no longer needed at all
rm -rf tools/kmp-impact-analyzer
```

Optionally delete the `gh-pages-history` branch:

```bash
git push origin --delete gh-pages-history
```

The target project compiles and tests exactly as it did before. No KMP-IMPACT residue remains anywhere else.

## See also

- [Installation](installation.md) — the actual commands, without the commentary.
- [Requirements](requirements.md) — what versions of JDK, Gradle, Kotlin, and KMP this works with.
- [Guides → How everything talks to each other](../guides/how-it-works.md) — the runtime data flow.
