# `kmp-impact detect-version-changes`

Diff two `libs.versions.toml` files. Used by the GitHub Action's `detect` job to feed `--dependency`, `--before-version`, and `--after-version` automatically.

## Synopsis

```bash
kmp-impact detect-version-changes \
  --before BASE.toml \
  --after  HEAD.toml \
  [--format json|github]
```

## Flags

| Flag | Required | Default | Description |
|---|---|---|---|
| `--before` | yes | — | Path to the BEFORE catalog (typically PR merge-base). |
| `--after` | yes | — | Path to the AFTER catalog (typically PR head). |
| `--format` | no | `json` | Output format: `json` (machine-readable) or `github` (GitHub Actions `KEY=VALUE` lines for the `$GITHUB_OUTPUT` file). |

## JSON output

```json
{
  "has_changes": true,
  "changes": [
    {
      "dependency_group": "io.ktor",
      "version_key": "ktor",
      "before_version": "2.3.8",
      "after_version": "2.3.11"
    }
  ]
}
```

When the catalogs are identical, `has_changes` is `false` and `changes` is `[]`. Exit code is always `0` on a successful diff — the absence of changes is data, not an error.

## `github` output

For consumption inside a GitHub Actions step that writes to `$GITHUB_OUTPUT`:

```text
has_changes=true
change_count=1
dependency_group=io.ktor
version_key=ktor
before_version=2.3.8
after_version=2.3.11
```

Only the first change is exposed as scalar variables; the reference workflow uses these to drive the downstream `static-pipeline` and `droidbot` jobs.

## Example

```bash
kmp-impact detect-version-changes \
  --before /tmp/base.libs.versions.toml \
  --after  /tmp/head.libs.versions.toml \
  --format github >> "$GITHUB_OUTPUT"
```

## See also

- [Guides → Configuring GitHub Actions](../../guides/github-actions.md) — how the `detect` job is wired in the reference workflow.
- [`VersionChange`](../contracts/index.md) — the Pydantic model behind each `changes[]` entry.
