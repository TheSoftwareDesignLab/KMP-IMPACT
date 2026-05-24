# Results

## Pipeline state across 53 PRs

| State | Count | Share | Meaning |
|---|---:|---:|---|
| `OK` | 44 | 83.0 % | HTML report published; PR comment posted. |
| `EXPECTED_SKIPPED` | 5 | 9.4 % | `pip/requests-*` PRs that do not touch the Gradle catalog. Skipped by design. |
| `BLOCKED_BUILD` | 2 | 3.8 % | `gradle-wrapper 8.x → 9.5.1` bumps incompatible with the project's AGP 8.x. Reported with explicit reason. |
| `BLOCKED_PAGES` | 2 | 3.8 % | Analysis completed; `actions/deploy-pages@v4` cancelled by repository-level concurrency. |

**Pipeline success rate (excluding `EXPECTED_SKIPPED`)**: `44 / 48 = 91.7 %`.
**Adjusted success rate (also excluding wrapper bumps that are intrinsically broken)**: `44 / 46 = 95.7 %`.

### Per-repository breakdown

| Repo | PRs | `OK` | `EXP_SKIP` | `BLOCK_BUILD` | `BLOCK_PAGES` |
|---|---:|---:|---:|---:|---:|
| Pokedex | 10 | 8 | 1 | 1 | 0 |
| Confetti | 11 | 8 | 1 | 0 | 2 |
| DroidconKotlin | 11 | 9 | 1 | 1 | 0 |
| KMP-App-Template | 10 | 9 | 1 | 0 | 0 |
| kmp-production-sample | 11 | 10 | 1 | 0 | 0 |

## Static analysis — Precision, Recall, F1

Mean over `library_kotlin` PRs with `pipeline = OK` (N = 25):

| Metric | Mean |
|---|---:|
| Precision | **0.615** |
| Recall | **0.720** |
| F1 | **0.658** |

### Per repository

| Repo | N effective | Precision | Recall | F1 |
|---|---:|---:|---:|---:|
| DroidconKotlin | 6 | 0.822 | 0.833 | **0.828** |
| kmp-production-sample | 5 | 0.642 | 1.000 | **0.780** |
| Confetti | 4 | 0.724 | 0.750 | **0.737** |
| Pokedex | 4 | 0.654 | 0.750 | **0.699** |
| KMP-App-Template | 6 | 0.288 | 0.333 | **0.307** |

### Qualitative reading

- **DroidconKotlin (0.828)** — Best F1. The bumped dependencies map cleanly into the analyzer's Maven → Kotlin table. Manual audit confirmed `ktor` 6 / 6 direct files with zero discrepancies.
- **kmp-production-sample (0.780)** — Perfect Recall. The analyzer never misses a file the ground truth marks as impacted. Precision is moderate because some bumps reach configuration files outside the strict import graph (AGP, `com.github.ben-manes.versions`).
- **Confetti (0.737)** — Largest tree (299 Kotlin files, 62 direct seeds in `io.insert-koin`). Solid F1 with no scalability surprises; phase-2 walltime stays under 90 s.
- **Pokedex (0.699)** — Mid-range. Notable miss: the analyzer does not currently map `io.github.qdsfdhvh` → `com.seiko.imageloader`, losing one direct file in the `image-loader` bump.
- **KMP-App-Template (0.307)** — Lowest F1. The bumped dependencies belong to `org.jetbrains.compose.*` (Compose Multiplatform), but the project imports them as `androidx.compose.*`. The analyzer does not yet resolve this correspondence — [**L7**](../troubleshooting.md#l7-compose-multiplatform-mapping). The ground truth resolves it manually; closing the gap is the single most impactful improvement available.

## Dynamic analysis — DroidBot

For every PR with `pipeline = OK`:

| Repo | PRs OK | APKs compiled | DroidBot ran | UTGs generated | Mean `screen_diffs` |
|---|---:|---:|---:|---:|---:|
| Pokedex | 8 | 8 / 8 | 8 / 8 | 8 / 8 | 0 |
| Confetti | 8 | 8 / 8 | 8 / 8 | 8 / 8 | 0 |
| DroidconKotlin | 9 | 9 / 9 | 9 / 9 | 9 / 9 | 0 |
| KMP-App-Template | 9 | 9 / 9 | 9 / 9 | 9 / 9 | 0 |
| kmp-production-sample | 10 | 10 / 10 | 10 / 10 | 10 / 10 | 0 |

100 % of OK PRs produced a real UTG (no legitimate `BLOCKED_DROIDBOT`). The mean `screen_diffs = 0` is consistent with the bumps under test being mostly patch / minor in libraries that do not alter user-visible navigation under the budgeted exploration. This is an *observation*, not a guarantee that no behavioural regression exists.

## Visualization — 5 criteria

| # | Criterion | Reports passing | Share |
|---|---|---:|---:|
| C1 | Chain `dependency → files → screens` is navigable from a single flow | 44 / 44 | 100 % |
| C2 | Direct vs. transitive impact distinguished; real BFS edges persisted | 44 / 44 | 100 % |
| C3 | DroidBot UTG embedded interactively, edges decorated by impact level | 44 / 44 | 100 % |
| C4 | CodeCharta with `area=rloc`, `height=mcc`, `color=impact_level` | 44 / 44 | 100 % |
| C5 | Risk label derived from counters, not hard-coded | 44 / 44 | 100 % |

## Limitations of the evaluation

1. **DroidBot replication.** Each PR ran DroidBot once per version due to runner-time cost. Reporting is per-observation, not per-mean. Compensation: 44 distinct PRs rather than a smaller set with multiple runs.
2. **Compose Multiplatform mapping (L7).** Lowers F1 on KMP-App-Template; the ground truth resolves the correspondence and the analyzer currently does not. The gap is closable in code; the metric will move with it.
3. **Package-level import resolution.** Both ground truth and analyzer resolve at package level (no full KSP). Symmetric, so it does not bias P / R / F1, but inflates absolute counts on both sides.
4. **Wrapper bumps (`gradle-wrapper 8.x → 9.5.1`).** Intrinsically broken on AGP 8.x. Reported as `BLOCKED_BUILD` with the Gradle log — the pipeline behaves correctly; there is just no HTML report to evaluate.
5. **Pages concurrency (L6).** Two PRs ended as `BLOCKED_PAGES` because GitHub Pages cancels concurrent deploys at the repository level. The analysis still completed; only the deploy was cancelled.

## Deliverables

| Artifact | Path |
|---|---|
| Per-PR inventory | `evaluacion/data/runs.json` |
| Per-PR metrics | `evaluacion/data/metrics.csv` |
| Per-repo aggregates | `evaluacion/data/metrics_summary.json` |
| Manual ground truth | `evaluacion/data/ground_truth/<repo>.json` |
| Audit log | `evaluacion/data/ground_truth_audit.json` |
| Figures | `evaluacion/figures/{pipeline_state_per_repo,prf1_per_repo,state_by_bump_category}.png` |

All artifacts live in the demonstration repository [`EstebanCastel/KMP-IMPACT-Pokedex-with-tool`](https://github.com/EstebanCastel/KMP-IMPACT-Pokedex-with-tool) under `evaluacion/`.
