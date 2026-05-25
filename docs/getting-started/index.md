# Getting Started

This section gets you from zero to a first analysis on your machine.

<div class="kmp-callout" markdown>
You'll need **Python 3.10+** and, for the dynamic phase, **JDK 21** and an Android emulator. Static-only analyses run anywhere Python does.
</div>

## In this section

<div class="kmp-grid" markdown>

<div class="card" markdown>
### Installation
Clone the repo, install the package in editable mode, run the test suite.

[Open →](installation.md)
</div>

<div class="card" markdown>
### Requirements
Host, target-repository, and dynamic-phase prerequisites — including Kotlin 2.x Compose Compiler caveats.

[Open →](requirements.md)
</div>

<div class="card" markdown>
### Project Structure
What lives in `src/`, `tests/`, `docs/`, `examples/` — a tour of the codebase.

[Open →](project-structure.md)
</div>

<div class="card" markdown>
### Quickstart
Run the pipeline end-to-end against a sample KMP repository and open the HTML report.

[Open →](quickstart.md)
</div>

</div>

## Next steps

Once you have a local analysis running, choose your path:

- **Wire it into CI** — [Guides → GitHub Actions](../guides/github-actions.md).
- **Understand the internals** — [Architecture → Pipeline overview](../architecture/pipeline-overview.md).
- **Look up a flag or a JSON field** — [API Reference](../reference/index.md).
