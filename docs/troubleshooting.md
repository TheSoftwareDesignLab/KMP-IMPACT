# Troubleshooting

The pipeline emits explicit failure reasons rather than silently degrading. This page lists every limitation acknowledged in the design, the symptom you will see, and the recommended diagnosis or workaround.

## L1 — Direct `build.gradle.kts` versions are not detected

**Symptom.** The pipeline runs, `Detect version change` reports `no Gradle catalog change found`, and the rest of the jobs are skipped.

**Cause.** The change detector only parses `gradle/libs.versions.toml`. If the bump lives directly in `build.gradle.kts`, `gradle.properties`, `pluginManagement { … }`, or `buildSrc/`, it is not detected.

**Workarounds.**

- Migrate the project to a Gradle Version Catalog. This is the Gradle-recommended layout since 2023.
- Or, run KMP-IMPACT locally with explicit `--before-version` / `--after-version` flags.

## L2 — `plugin_or_toolchain` bumps report zero static impact

**Symptom.** The report opens correctly but lists `0 impacted files` for bumps of `com.android.application`, `com.android.library`, `com.google.devtools.ksp`, the Gradle wrapper, etc.

**Cause.** The static phase looks at Kotlin `import` statements. Build plugins operate at the Gradle level — they leave no trace in `.kt` files — so the BFS legitimately finds nothing.

**This is intentional.** The bump is still detected, the version diff is still reported, and the dynamic phase still runs. The PR comment shows the bump category to signal that the static section is empty by design.

## L3 — Non-convention source-set layout

**Symptom.** Files appear in the report with `source_set = "unknown"`. `expect`/`actual` pairs from those files are not resolved.

**Cause.** The analyzer derives the source set from the path: `src/<sourceSet>Main/kotlin/`. Modules that place Kotlin sources outside this convention (e.g., `src/main/kotlin/` only, or convention plugins with custom directories) hit this branch.

**Workarounds.**

- Move sources under the conventional layout — almost every KMP project ships with it by default.
- Or, parse `settings.gradle.kts` to obtain the actual module → directory map (planned; see roadmap on GitHub Discussions).

## L4 — DroidBot `BLOCKED` (no APK) {#l4-droidbot-blocked-no-apk}

**Symptom.** The report shows `Dynamic analysis: BLOCKED — DroidBot produced no UTG artifact` or the workflow's `Run DroidBot on emulator` step says `No APKs produced`.

**Cause.** One of the following:

1. `Build BEFORE APK` or `Build AFTER APK` failed. Open the Gradle log; common causes: incompatible Gradle wrapper, missing Compose Compiler plugin under Kotlin 2.x, or wrong `jvmToolchain(…)` placement.
2. The Android module was not detected. Symptom: `Detect Android app module` prints `none found`.
3. The detected module is a `com.android.library` rather than a `com.android.application` and therefore does not produce an APK.

**Triage order.**

1. Check `Detect Android app module` — must list `:android`, `:app`, `:composeApp`, or similar.
2. Check `Ensure Gradle wrapper is recent enough for AGP` — must show ≥ 8.7 for AGP 8.x and ≥ 9.0 for AGP 9.x.
3. Check `Build BEFORE APK` — must end with `BEFORE APK: /tmp/before.apk`.
4. Check `Build AFTER APK` — same expectation.
5. If all four are green, confirm that `phase3/before-utg/` and `phase3/after-utg/` contain `utg.js` or `utg.json` — empty directories are rejected by the merge step on purpose.

## L5 — CodeCharta viewer requires embedded assets

**Symptom.** The CodeCharta section of the report shows a static SVG instead of the interactive 3D viewer.

**Cause.** The interactive viewer is a 15 MB Angular bundle and is not produced on the fly. KMP-IMPACT ships the SVG fallback by default and looks for the viewer assets at `evidence/codecharta/codecharta-viewer/` when the user wants the embedded experience.

**Workarounds.**

- Open the same `phase5/impact.cc.json` in the public [CodeCharta visualization](https://maibornwolff.github.io/codecharta/visualization/app/index.html).
- Or commit the viewer bundle to your repo under `evidence/codecharta/codecharta-viewer/`.

## L6 — `deploy-pages` cancellation {#l6-deploy-pages-cancellation}

**Symptom.** The `Deploy report to GitHub Pages` job ends as `cancelled` even though the analysis completed. The PR comment contains a link that 404s.

**Cause.** `actions/deploy-pages@v4` enforces concurrency at the repository level (one active deployment at a time). When several Dependabot PRs finish nearly simultaneously, the newer deploy cancels the older one.

**Workarounds.**

- The reference workflow already sets `concurrency: { group: pages-${{ github.ref }}, cancel-in-progress: false }`. This bounds the conflict to *across-PR* contention.
- For multi-PR bursts, re-run cancelled deploys serially with `gh run watch` between calls.
- For full evaluations across many repos under the same owner, stagger Dependabot's daily schedule or use a self-hosted runner queue.

## L7 — Compose Multiplatform mapping {#l7-compose-multiplatform-mapping}

**Symptom.** The static phase under-reports impact for projects that depend on `org.jetbrains.compose.*` Maven coordinates. Recall drops for bumps such as `compose-material 1.6.x → 1.7.x` on Compose Multiplatform projects.

**Cause.** The Maven → Kotlin mapping table does not yet resolve `org.jetbrains.compose.*` to the package root `androidx.compose.*` that Compose Multiplatform reuses for its source-level API.

**Status.** Closable in [`MAVEN_TO_KOTLIN`](https://github.com/EstebanCastel/KMP-IMPACT-Reviewing-Dependency-Updates-in-Kotlin-Multiplatform/blob/main/src/kmp_impact_analyzer/phase2_static/dependency_mapping.py). Pull requests welcome.

## L8 — Dependency injection misses

**Symptom.** Files that use a dependency only through DI bindings (Koin `single { … }`, Hilt `@Provides`, runtime reflection) are not flagged as impacted.

**Cause.** The static phase reads `import` declarations. Bindings expressed without an `import` for the dependency root are invisible to the BFS.

**Workarounds.**

- The transitive BFS still captures files that *use* the binding via an explicit Koin/Hilt API, so the practical miss is bounded.
- Future work: a DI-aware companion pass that parses `module { … }` blocks.

## Upstream workflows cluttering the PR UI

**Symptom.** Each PR shows multiple red checks even though `impact-analysis.yml` is green. Examples in real evaluations: Backend Deploy, iOS CI, Wear CI, publish-android.

**Cause.** Forks of upstream KMP projects inherit workflows that depend on secrets you do not own (Google Cloud, App Store Connect, Firebase, signing keystores). Those workflows fail before doing useful work.

**Fix.** Disable every workflow other than `impact-analysis.yml` once per repository:

```bash
gh api repos/<owner>/<repo>/actions/workflows --jq \
  '.workflows[] | select(.path | startswith(".github/workflows/")) |
   select(.path != ".github/workflows/impact-analysis.yml") | .id' \
| while read wid; do
    gh api -X PUT repos/<owner>/<repo>/actions/workflows/$wid/disable
  done
```

This does **not** delete the workflow files; it just prevents them from triggering. New PRs will only show the KMP-IMPACT checks.

## "DroidBot skipped" on old PRs

**Symptom.** A PR opened before the analyzer was wired up still shows `Dynamic analysis: skipped` after the workflow was added.

**Cause.** Dependabot's branch base is the `main` of the moment the PR was opened. If `main` later gained the workflow, the old PR's `merge-base` is still the old commit without the workflow.

**Fix.** `@dependabot rebase` (or `@dependabot recreate`) on the old PR — it rebuilds the branch on top of current `main` and re-runs CI.
