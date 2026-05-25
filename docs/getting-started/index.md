# Getting Started

This section gets you from zero to a first analysis on your machine, and tells you exactly what the install pulls in so there are no surprises.

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
Host machine and target-repository prerequisites — plus the compatibility matrix of Kotlin / AGP / Gradle versions validated end-to-end.

[Open →](requirements.md)
</div>

<div class="card" markdown>
### What gets installed
Line-by-line walkthrough of what every command in the install actually downloads, creates, or modifies. Both locally and inside a CI runner.

[Open →](what-gets-installed.md)
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

- **Understand the runtime flow** — [Guides → How everything talks to each other](../guides/how-it-works.md).
- **Wire it into CI** — [Guides → Configuring GitHub Actions](../guides/github-actions.md).
- **Understand the internals** — [Architecture → Pipeline overview](../architecture/pipeline-overview.md).
- **Look up a flag or a JSON field** — [API Reference](../reference/index.md).
