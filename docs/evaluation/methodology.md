# Methodology

## Experimental design

Five Kotlin Multiplatform projects, ten repositories. For each project, two GitHub repositories with **identical** `gradle/libs.versions.toml` and `.github/dependabot.yml`:

- `KMP-IMPACT-<project>-with-tool` — keeps the KMP-IMPACT pipeline and publishes reports to GitHub Pages.
- `KMP-IMPACT-<project>-baseline` — same code, but without the `tools/kmp-impact-analyzer/`, the `impact-analysis.yml` workflow, or the `evidence/` directory. Used as the source of truth for ground-truth construction.

Both sides receive the same Dependabot PRs naturally. The `-with-tool` side runs the pipeline; the `-baseline` side serves as the unmodified reference for inspection.

## Ground-truth construction

For every Dependabot PR, the ground-truth set is built by:

1. **Catalog diff** — parse `gradle/libs.versions.toml` BEFORE and AFTER. Map the changed Maven group to a Kotlin import root via [`MAVEN_TO_KOTLIN`](https://github.com/EstebanCastel/KMP-IMPACT-Pokedex-with-tool/blob/main/evaluacion/scripts/build_ground_truth.py).
2. **Direct seeds** — `grep -lrE '^import <prefix>' --include='*.kt' .` over the baseline repo at the merge-base commit.
3. **Transitive set** — BFS over project-internal imports starting from each direct seed.
4. **Manual audit** — for every direct seed, open the `.kt` file and read the `import` line. Cross-check the seed set against an independent grep.

## Audit results

| Check | Result |
|---|---|
| Cross-grep of the direct set per dependency | 121 / 121 dependencies — 0 differences against the GT |
| Each transitive file imports something in `direct ∪ transitive` | 0 / 121 orphans |
| Disjointness `direct ∩ transitive = ∅` | 0 violations |
| `|direct| + |transitive| ≤ total_kt_files` | 0 over-counts |
| Manual file-by-file audit of direct seeds | 89 / 89 verified |

Per-repository manual audit highlights:

| Repo | Key dependency | Direct files audited | Cross-grep |
|---|---|---:|---|
| Pokedex | `io.insert-koin` | 13 / 13 | 13 OK |
| Confetti | `io.insert-koin` | 5 sampled / 62 | 62 OK |
| DroidconKotlin | `io.ktor` | 6 / 6 | 6 OK |
| KMP-App-Template | `io.ktor` | 2 / 2 | 2 OK |
| kmp-production-sample | `io.ktor` | 6 / 6 | 6 OK |

Both the analyzer and the ground truth use the same import-resolution heuristic (package-level, no KSP). When a package contains several files and an external import targets one of them, the others may appear "reachable" through the BFS. This is symmetric across analyzer and GT and therefore does not bias P/R/F1 — but it does inflate absolute counts on both sides.

## Static-analysis metrics

Given the automatic set `A = phase4/consolidated.json.impacted_files` and the manual set `M = ground_truth.yml.direct ∪ transitive`:

```text
Precision = |A ∩ M| / |A|
Recall    = |A ∩ M| / |M|
F1        = 2 · P · R / (P + R)
```

PRs with `M = A = ∅` (dependencies declared but never imported in any `.kt`) are excluded from the mean, since they would inflate metrics by appearing as trivially correct.

## Dynamic-analysis protocol

For every PR that reached `pipeline = OK`:

1. Build the Debug APK on the BEFORE state. Fail explicitly if the APK is not produced.
2. Build the Debug APK on the AFTER state. Same fail-fast rule.
3. Launch DroidBot inside `reactivecircus/android-emulator-runner` (Android 33, x86_64) with `policy = dfs_greedy`, `count = 100`, `timeout = 90`, `-grant_perm`, `-is_emulator`.
4. Diff the resulting UTGs at the state level.

**Limitation acknowledged.** Each PR ran DroidBot once per APK (BEFORE and AFTER) due to runner-time cost. The reported `screen_diffs` is therefore a single observation per version, not a multi-run mean ± standard deviation. The evaluation compensates by reporting more PRs (44) rather than more runs per PR.

## Visualization criteria

Each report was rated against five criteria, evaluated by inspecting the published HTML:

| # | Criterion |
|---|---|
| C1 | The report makes the chain *dependency → impacted files → impacted screens* visible from a single navigation flow. |
| C2 | Direct and transitive impact are visually distinguishable in the propagation graph, and BFS edges are persisted (not redrawn). |
| C3 | The DroidBot UTG is embedded interactively (iframe with Cytoscape), with edges decorated by impact level. |
| C4 | CodeCharta is embedded with `area = rloc`, `height = mcc`, `color = impact_level`. |
| C5 | The risk label is computed from the static and dynamic counters; values are not hard-coded. |

## Reproducibility

Scripts and data are open. To rebuild the evaluation from scratch:

```bash
# 1. Build ground truth from each baseline
for r in evaluacion/repos/KMP-IMPACT-*-baseline; do
  out="evaluacion/data/ground_truth/$(basename $r).json"
  python3 evaluacion/scripts/build_ground_truth.py "$r" "$out"
done

# 2. Audit the ground truth
python3 evaluacion/scripts/audit_ground_truth.py

# 3. Collect reports from each -with-tool repo
python3 evaluacion/scripts/collect_reports.py

# 4. Compute metrics
python3 evaluacion/scripts/compute_metrics.py

# 5. Render figures and LaTeX tables
python3 evaluacion/scripts/render_figures.py
```

The full scripts live in [`evaluacion/scripts/`](https://github.com/EstebanCastel/KMP-IMPACT-Pokedex-with-tool/tree/main/evaluacion/scripts) of the demonstration repository.
