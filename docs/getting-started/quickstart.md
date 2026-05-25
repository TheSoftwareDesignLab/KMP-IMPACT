# Quickstart

End-to-end run on a Kotlin Multiplatform project in under five minutes. We will analyse a Ktor 2.3.8 → 2.3.11 bump on a sample project and open the HTML report.

## 1. Pick a project

Any KMP project that:

- keeps its versions in `gradle/libs.versions.toml`,
- has at least one Kotlin source file that imports the dependency you want to bump,
- builds with the Gradle wrapper.

For this walkthrough we'll use the demo Pokedex KMP app:

```bash
git clone https://github.com/EstebanCastel/KMP-IMPACT-Pokedex-with-tool.git ~/pokedex-kmp
```

## 2. Run a static analysis

```bash
kmp-impact analyze \
  --repo ~/pokedex-kmp \
  --dependency io.ktor \
  --before-version 2.3.8 \
  --after-version 2.3.11 \
  --output-dir output \
  --skip-dynamic
```

The first run is the slowest because Gradle resolves dependencies for both BEFORE and AFTER shadow copies. Subsequent runs reuse the Gradle cache.

## 3. Open the report

```bash
open output/report/index.html         # macOS
xdg-open output/report/index.html     # Linux
start  output/report/index.html       # Windows
```

You will see six tabs:

| Tab | What it shows |
|---|---|
| **Summary** | Bump, impact counts, risk level, recommendation. |
| **Static impact** | List of `.kt` files marked direct (`level 2`) or transitive (`level 1`). |
| **UI impact** | DroidBot UTG diff. Empty when `--skip-dynamic`. |
| **Traceability** | File → Screen mapping with sticky header. |
| **Visualization** | Propagation tree (BFS, interactive) and CodeCharta JSON download. |
| **Raw artifacts** | Links to `phase2/impact_graph.json`, `phase4/consolidated.json`, etc. |

## 4. (Optional) Add the dynamic phase

Re-run without `--skip-dynamic`, with the Android SDK and an emulator available:

```bash
kmp-impact analyze \
  --repo ~/pokedex-kmp \
  --dependency io.ktor \
  --before-version 2.3.8 \
  --after-version 2.3.11 \
  --output-dir output-full
```

The pipeline will:

1. Build `:android:assembleDebug` on the BEFORE state.
2. Build the same target on the AFTER state.
3. Launch DroidBot against each APK and diff the resulting UTGs.

If either APK fails to build, the report will mark the dynamic phase `BLOCKED` with the compilation log attached. The static report is still produced.

## 5. Where to go next

- Adapt your own KMP repo with the [Preparing an existing KMP project](../guides/preparing-a-kmp-project.md) checklist before wiring it up.
- Plug it into a real PR flow with the [GitHub Action guide](../guides/github-actions.md).
- Read the [Pipeline overview](../architecture/pipeline-overview.md) to understand what each phase contributes.
- Browse the [Presets](../presets/index.md) for ready-to-fork examples per KMP project.
