# DroidconKotlin

The Droidcon conference app. Best static-analysis F1 in the evaluation (0.828).

- Reference repo: [`EstebanCastel/KMP-IMPACT-DroidconKotlin-with-tool`](https://github.com/EstebanCastel/KMP-IMPACT-DroidconKotlin-with-tool)
- Baseline: [`EstebanCastel/KMP-IMPACT-DroidconKotlin-baseline`](https://github.com/EstebanCastel/KMP-IMPACT-DroidconKotlin-baseline)

## Stack

| Component | Version |
|---|---|
| Gradle wrapper | 8.14.3 |
| Kotlin | 2.2.21 |
| Compose Compiler plugin | applied |
| Targets | Android, iOS, shared (commonMain, androidMain, iosMain) |

## Upstream workflows — disable

Two workflows to disable: `build.yml` (publishes to internal Google Cloud) and `gradle-wrapper.yaml` (uses a custom SCM token).

```bash
gh api -X PUT repos/<owner>/<repo>/actions/workflows/<build-yml-id>/disable
gh api -X PUT repos/<owner>/<repo>/actions/workflows/<wrapper-yml-id>/disable
```

After disabling, only `impact-analysis.yml` runs on PRs.

## Why this preset is a good fit

DroidconKotlin's dependencies map cleanly into the analyzer's Maven → Kotlin table — `io.ktor:*` resolves to `io.ktor.*`, `io.insert-koin:*` to `org.koin.*` — and the project follows the conventional KMP source-set layout exactly. If your repo looks like DroidconKotlin, the integration is essentially copy-paste.
