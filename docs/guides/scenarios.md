# Writing Scenarios

Scenarios are reproducible bundles: a Kotlin Multiplatform project, one or more version bumps to analyse, and an optional ground truth. They are the input format consumed by `kmp-impact run-scenario` and the recommended way to benchmark changes to the analyzer itself.

## Layout

```text
my_scenario/
├── scenario.yml
├── ground_truth.yml   # optional
├── README.md
└── repo/              # optional: vendored copy of the KMP project
```

## `scenario.yml`

```yaml
name: pokedex_ktor_minor
description: "Bump io.ktor 2.3.8 -> 2.3.11 on Pokedex KMP."
repo:
  path: ./repo            # or a git URL + sha
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

The ground truth is intentionally minimal:

- **Direct files** import the bumped library (`^import io.ktor.*`).
- **Transitive files** are reached by the static BFS from a direct file.
- **Screens** are matched by Compose composable name against DroidBot UTG state labels.

## Running a scenario

```bash
kmp-impact run-scenario \
  --scenario-dir scenarios/pokedex_ktor_minor \
  --output-dir output-pokedex \
  --skip-dynamic
```

When the scenario provides a `ground_truth.yml`, the report includes a metrics card with Precision / Recall / F1 for files and screens.

## Building a ground truth from scratch

Workflow for auditing a new repository:

1. Run `kmp-impact analyze` once with `--skip-dynamic` to collect a candidate impact list.
2. Open each candidate file. Confirm the import against the bumped dependency.
3. Record the verified set in `direct_files` / `transitive_files` under `ground_truth.yml`.
4. Re-run the scenario; the metrics now reflect the cleaned ground truth.

## See also

- [API Reference → `run-scenario`](../reference/cli/run-scenario.md) — every flag.
- [API Reference → `evaluate`](../reference/cli/evaluate.md) — comparing a result against a ground truth offline.
