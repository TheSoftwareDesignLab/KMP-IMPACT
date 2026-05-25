# Phase 3 — Dynamic analysis (DroidBot)

The third phase compares the UI navigation graph of the BEFORE and AFTER APKs. It detects screens that exist in only one state and edges that appear or disappear, labelling them as `regression`, `addition`, or `no-change`.

## Inputs

- `phase1/before/`, `phase1/after/` — shadow copies from Phase 1.
- A running Android emulator (locally or via `reactivecircus/android-emulator-runner` in CI).

## Outputs

```text
phase3/
├── before.utg/             # DroidBot UTG, BEFORE APK
├── after.utg/              # DroidBot UTG, AFTER APK
└── ui_regressions.json
```

## What happens

1. Assemble the debug APK on the BEFORE shadow copy with `./gradlew :<android-module>:assembleDebug`. If this fails, mark Phase 3 `BLOCKED — APK assembly failed (BEFORE)` and stop.
2. Same for AFTER.
3. Install the BEFORE APK on the emulator. Run DroidBot for the configured budget — defaults `count = 100`, `timeout = 90`, `policy = dfs_greedy`, `-grant_perm`, `-is_emulator`. Persist the UTG to `phase3/before.utg/`.
4. Uninstall the BEFORE APK, install AFTER. Repeat DroidBot. Persist to `phase3/after.utg/`.
5. Diff the two UTGs at the state level. State identity is based on the activity name plus the set of clickable element IDs — DroidBot's default heuristic.

## Output sample

```json
{
  "status": "completed",
  "blocked_reason": "",
  "before_screens": ["MainActivity", "DetailActivity"],
  "after_screens":  ["MainActivity", "DetailActivity"],
  "diffs": [],
  "activity_coverage_before": { "MainActivity": 18, "DetailActivity": 6 },
  "activity_coverage_after":  { "MainActivity": 17, "DetailActivity": 7 },
  "nodes_before": 14, "nodes_after": 14,
  "structures_before": 6, "structures_after": 6
}
```

On failure the same model carries a `blocked` status and an empty payload:

```json
{
  "status": "blocked",
  "blocked_reason": "DroidBot produced no UTG artifact",
  "before_screens": [],
  "after_screens": [],
  "diffs": []
}
```

## Edge cases

- **Build failure on BEFORE or AFTER.** Most common cause is Kotlin 2.x without the Compose Compiler plugin. See [L4](../troubleshooting.md#l4-droidbot-blocked-no-apk).
- **Android module not detected.** The workflow probes the module names listed in [Reference → GitHub Action](../reference/github-action.md#android-module-autodetection). If none match, expose your real module under one of those aliases.
- **Empty UTG.** The merge step rejects empty UTG folders on purpose — an empty artifact would mask a real failure.
- **Emulator unavailable.** Locally, fall back to `--skip-dynamic`. In CI, the emulator is provided by `reactivecircus/android-emulator-runner`.

## Contracts

- [`UIRegressions`](../reference/contracts/ui-regressions.md)
- [`DynamicStatus`](../reference/contracts/ui-regressions.md#dynamicstatus)

## Next phase

Phase 4 reads `phase2/impact_graph.json` and `phase3/ui_regressions.json` and produces the canonical `consolidated.json`.

→ [Phase 4 — Consolidation](phase-4-consolidation.md)
