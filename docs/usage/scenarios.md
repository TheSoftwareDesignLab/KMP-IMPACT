# Scenarios

Scenarios are reproducible bundles: a Kotlin Multiplatform project, one or more version bumps to analyse, and the expected ground truth. They are the basis of the [empirical evaluation](../evaluation/results.md) and the recommended way to benchmark changes to the analyzer itself.

## Layout

```text
my_scenario/
├── scenario.yml
├── ground_truth.yml
├── README.md
└── repo/                  # optional: vendored copy of the KMP project
```

## `scenario.yml`

```yaml
name: pokedex_ktor_minor
description: "Bump io.ktor 2.3.8 → 2.3.11 on Pokedex KMP."
repo:
  path: ./repo              # or a git URL + sha
bumps:
  - dependency: io.ktor
    before: 2.3.8
    after:  2.3.11
analysis:
  skip_dynamic: false
  output_dir: output
```

Multiple bumps can be analysed in the same scenario; each is reported in its own `phaseN/` subdirectory.

## `ground_truth.yml`

```yaml
direct_files:
  - shared/src/commonMain/kotlin/com/example/api/PokedexClient.kt
  - shared/src/commonMain/kotlin/com/example/data/PokemonRepository.kt
transitive_files:
  - shared/src/commonMain/kotlin/com/example/ui/PokemonViewModel.kt
ui_screens:
  - PokemonListScreen
  - PokemonDetailScreen
```

The ground truth is intentionally minimal. Direct files are those that import the bumped library; transitive files are those reached by the static BFS from a direct file. Screens are by Compose composable name, matched against DroidBot's UTG state labels.

## Running a scenario

```bash
kmp-impact run-scenario \
  --scenario-dir scenarios/pokedex_ktor_minor \
  --output-dir output-pokedex \
  --skip-dynamic
```

When the scenario provides a `ground_truth.yml`, the report includes a Metrics card with Precision / Recall / F1 for both files and screens.

## Building a ground truth from scratch

If you are auditing a new repository, the recommended workflow is:

1. Run `kmp-impact analyze` once with `--skip-dynamic` to collect a candidate impact list.
2. Open each candidate file and verify the import against the bumped dependency — note the verdict.
3. Save the verified set as `direct_files` / `transitive_files` in `ground_truth.yml`.
4. Re-run the scenario; the metrics will reflect the cleaned ground truth.

The thesis uses the [`scripts/build_ground_truth.py`](https://github.com/EstebanCastel/KMP-IMPACT-Pokedex-with-tool/blob/main/evaluacion/scripts/build_ground_truth.py) helper to bootstrap step 1 across multiple baseline repositories.
