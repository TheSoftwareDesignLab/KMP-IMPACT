# `kmp-impact run-scenario`

Run an analysis from a predefined scenario file. The scenario captures a single dependency bump together with its repository pointer; the runner reads `scenario.yml` and forwards the values to the same pipeline that powers [`analyze`](analyze.md).

## Synopsis

```bash
kmp-impact run-scenario \
  --scenario-dir DIR \
  [--output-dir OUT] \
  [--skip-dynamic]
```

## Flags

| Flag | Required | Default | Description |
|---|---|---|---|
| `--scenario-dir` | yes | — | Path to a directory that contains a `scenario.yml` file. |
| `--output-dir` | no | `output` | Destination for `phaseN/` and the HTML report. |
| `--skip-dynamic` | no | off | Skip Phase 3 (DroidBot). |

## `scenario.yml` schema

```yaml
# Required
dependency_group: io.ktor
before_version: "2.3.8"
after_version: "2.3.11"

# Required — exactly one of:
repo_path: ./repo          # local path
# repo_url: https://github.com/<owner>/<repo>.git
```

The runner ignores unknown keys. Sibling files in the same directory (`ground_truth.yml`, `README.md`) are not read by the CLI — the [`evaluate`](evaluate.md) command consumes `ground_truth.yml` separately.

## Authoring a scenario

```bash
mkdir -p scenarios/pokedex_ktor_minor
cat > scenarios/pokedex_ktor_minor/scenario.yml <<'YAML'
dependency_group: io.ktor
before_version: "2.3.8"
after_version: "2.3.11"
repo_path: ../../pokedex-kmp
YAML
```

For the authoring workflow and `ground_truth.yml` companion file, see [Guides → Writing Scenarios](../../guides/scenarios.md).

## Example

```bash
kmp-impact run-scenario \
  --scenario-dir scenarios/pokedex_ktor_minor \
  --output-dir output-pokedex \
  --skip-dynamic
```

## See also

- [`analyze`](analyze.md) — equivalent ad-hoc invocation with explicit flags.
- [`evaluate`](evaluate.md) — score a result against `ground_truth.yml` offline.
