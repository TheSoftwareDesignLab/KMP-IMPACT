# Evaluation

KMP-IMPACT was evaluated on **53 real Dependabot pull requests** opened automatically across 5 Kotlin Multiplatform projects, against a manually audited ground truth. No PR was hand-crafted; the experimental set is exactly what Dependabot produced under the [bias configuration](../usage/github-action.md#recommended-dependabot-configuration) applied to each repository.

## Setup at a glance

- **5 projects** chosen for stack diversity: Pokedex, Confetti, DroidconKotlin, KMP-App-Template, kmp-production-sample.
- **10 repositories** published under [`EstebanCastel/`](https://github.com/EstebanCastel) — one `-with-tool` and one `-baseline` per project — sharing the same `gradle/libs.versions.toml` and `.github/dependabot.yml`.
- **53 PRs** open by Dependabot, distributed as:
  - 33 `library_kotlin` bumps (Kotlin libraries with importable APIs).
  - 15 `plugin_or_toolchain` bumps (Kotlin, AGP, Compose Compiler, KSP, gradle-wrapper).
  - 5 `non_kotlin` bumps (`pip/requests` in the pipeline's Python helper).

## Three measured dimensions

| Dimension | Metric | Headline |
|---|---|---|
| Static analysis | Precision / Recall / F1 vs. manual ground truth | **F1 = 0.658**, Recall = **0.720** (mean across `library_kotlin` PRs, N=25) |
| Dynamic analysis | UTGs generated, screen diffs observed | **44/44** OK PRs produced a real UTG; mean `screen_diffs = 0` under the budgeted exploration |
| Visualization | 5 qualitative criteria across 44 published reports | **5/5** criteria satisfied on 100% of reports |

Plus an end-to-end **pipeline success rate**: **44/53** PRs reached the published HTML report; **44/48** when excluding `EXPECTED_SKIPPED` (`pip/*` PRs); **44/46** when also excluding `BLOCKED_BUILD` wrapper bumps that are intrinsically broken (the adjusted success rate of **95.7 %**).

## Where to read more

- [Methodology](methodology.md) — protocols, ground-truth construction, and audit procedure.
- [Results](results.md) — per-PR table, per-repo breakdown, and limitations of the evaluation itself.
- Ground-truth audit detail in the project repo at [`AUDITORIA_GROUND_TRUTH.md`](https://github.com/EstebanCastel/KMP-IMPACT-Pokedex-with-tool/blob/main/AUDITORIA_GROUND_TRUTH.md).
