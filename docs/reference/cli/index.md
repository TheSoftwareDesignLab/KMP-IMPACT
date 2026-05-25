# CLI

The package installs a single console entry point — `kmp-impact` — with four subcommands.

```bash
kmp-impact [COMMAND] [OPTIONS]
```

## Commands

| Command | Purpose |
|---|---|
| [`analyze`](analyze.md) | Run the full pipeline against a single dependency bump. |
| [`run-scenario`](run-scenario.md) | Run a YAML-defined scenario; supports multiple bumps and ground-truth comparison. |
| [`detect-version-changes`](detect-version-changes.md) | Diff two `libs.versions.toml` files. Used by the GitHub Action's `detect` job. |
| [`evaluate`](evaluate.md) | Compute Precision / Recall / F1 against a ground truth offline. |

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Pipeline produced a report. The report may still mark phases as `BLOCKED`. |
| `1` | CLI usage error (missing flag, invalid path). |
| `2` | Unrecoverable pipeline error (uncaught exception). Stack trace on stderr. |

A `BLOCKED` phase is **not** an error: the analyzer correctly detected that it could not produce real data for that phase. See [Diagnosing a BLOCKED phase](../../guides/diagnosing-blocked.md).
