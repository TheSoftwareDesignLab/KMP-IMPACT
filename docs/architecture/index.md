# Architecture

KMP-IMPACT is structured as a **5-phase pipeline**. Each phase reads a typed contract from the previous one and writes its own. There is no implicit shared state: every artifact is JSON on disk under `output/phaseN/`, which makes the pipeline reproducible, debuggable and trivially parallelizable across PRs.

- **[Pipeline overview](overview.md)** — the five phases at a glance and how they compose.
- **[Phase contracts](phases.md)** — inputs and outputs of every phase, edge cases included.
- **[Data contracts](contracts.md)** — the Pydantic v2 models that travel between phases.
