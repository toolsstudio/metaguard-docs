# CLI & CI Integration

MetaGuard exposes a headless CLI entry point for Unity batch mode. It runs the full scan, analyze, and report pipeline without opening the Editor UI — suitable for any CI environment that can invoke Unity.

---

## Command

```bash
Unity -batchmode \
  -projectPath /path/to/project \
  -executeMethod ToolsStudio.MetaGuard.Editor.CLI.MetaGuardCLI.Run \
  -logFile metaguard.log \
  -quit \
  -- --format json --threshold 4
```

Arguments after `--` are passed to MetaGuard. Unity batch mode arguments (`-batchmode`, `-projectPath`, etc.) are passed to Unity itself and must appear before `--`.

---

## Arguments

| Argument | Default | Description |
|---|---|---|
| `--policy <path>` | Auto-detected | Path to `metaguard_policy.json`. Defaults to `Assets/MetaGuard/metaguard_policy.json` in the project. Use this to point at a policy file in a non-standard location. |
| `--format json\|text` | `json` | Report output format. `json` produces a structured machine-readable report; `text` produces human-readable console output. |
| `--output <path>` | Auto | Write the report to a specific file path. When omitted, the report is written automatically to `MetaGuardReports/` at the project root. |
| `--threshold <0–5>` | `5` | Minimum risk level that contributes to the violation check. `5` = most lenient, `0` = strictest. Lower values produce stricter gates. |
| `--version` | — | Print MetaGuard version and exit with code 0. |
| `--help` | — | Print argument usage and exit with code 0. |

---

## Exit Codes

| Code | Meaning |
|---|---|
| `0` | Clean — no violations found |
| `1` | Violations found |
| `2` | Scan failed — corrupt project, missing files, Unity license issue, or internal error |

Exit code `2` indicates a failure at the scan level, not a GUID integrity issue. The report will contain an `error` field with details when exit code 2 is produced.

### Violation Conditions

Exit code `1` is produced when **any** of the following are true:

- `critical > 0` — at least one Critical-risk issue detected
- `high > 0` — at least one High-risk issue detected
- `health_score == 0` — always a violation regardless of other counts
- `health_score <= threshold` — health score at or below the configured threshold
- A `Block` policy rule matches any detected issue

---

## Auto-Report

When `--output` is not specified, MetaGuard writes the report automatically:

```
MetaGuardReports/report_YYYY-MM-DD_HH-mm-ss.json
```

The `MetaGuardReports/` directory is created at the project root (alongside `Assets/`) if it does not exist. This directory should be added to `.gitignore`.

The log line confirming the write:

```
[MetaGuardCLI] Report saved to: MetaGuardReports/report_2026-04-30_17-01-20.json
```

---

## Report Format (JSON)

```json
{
  "metaguard_version": "2.0.0",
  "timestamp": "2026-04-30T17:01:20.000Z",
  "has_violations": false,
  "health_score": 98,
  "health_score_raw": 98,
  "policy_active": true,
  "threshold": 4,
  "summary": {
    "total_issues": 2,
    "critical": 0,
    "high": 0,
    "medium": 0,
    "low": 2,
    "files_scanned": 34,
    "meta_files": 18
  },
  "issues": [
    {
      "type": "OrphanedMeta",
      "risk": "Low",
      "policy_action": "AutoFix",
      "path": "Assets/SomePath/leftover.meta",
      "guid": "",
      "references": 0,
      "suggested_fix": "DeleteOrphanedMeta",
      "description": "Orphaned .meta file: 'leftover.meta' has no matching asset on disk."
    }
  ]
}
```

`has_violations` is the authoritative pass/fail field. Use it in CI decision logic.

---

## Debug Line

After every violation evaluation, MetaGuard emits the following line to both stderr and the Unity `-logFile`:

```
ViolationCheck => critical:0 high:0 threshold:4 result:false
```

This is emitted regardless of the outcome. Use it to debug threshold and policy configuration when a CI run produces an unexpected pass or fail.

---

## Shell Script Template

A pre-configured shell script is included at `Assets/CI/metaguard_scan.sh`. It sets sensible defaults and handles the Unity path and project path as positional arguments.

```bash
# Run with defaults
./Assets/CI/metaguard_scan.sh /path/to/unity /path/to/project

# Override threshold via environment variable
RISK_THRESHOLD=2 ./Assets/CI/metaguard_scan.sh /path/to/unity /path/to/project
```

The script uses `--output` (file-write mode) rather than stdout redirect, which prevents Unity's batch-mode log output from interleaving with the JSON report.

---

## GitHub Actions

```yaml
name: MetaGuard GUID Integrity Check

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  metaguard:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Cache Unity Library
        uses: actions/cache@v4
        with:
          path: Library
          key: Library-${{ hashFiles('Assets/**', 'Packages/**', 'ProjectSettings/**') }}
          restore-keys: Library-

      - name: Run MetaGuard Scan
        uses: game-ci/unity-builder@v4
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
        with:
          unityVersion: 6000.0.10f1
          buildMethod: ToolsStudio.MetaGuard.Editor.CLI.MetaGuardCLI.Run
          customParameters: >-
            -logFile MetaGuardReports/metaguard.log
            -- --format json --threshold 4

      - name: Upload Report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: metaguard-report
          path: MetaGuardReports/
```

---

## GitLab CI

```yaml
metaguard:
  stage: test
  script:
    - >
      /opt/Unity/Editor/Unity
      -batchmode
      -nographics
      -projectPath "$CI_PROJECT_DIR"
      -executeMethod ToolsStudio.MetaGuard.Editor.CLI.MetaGuardCLI.Run
      -logFile "$CI_PROJECT_DIR/MetaGuardReports/metaguard.log"
      -quit
      -- --format json --threshold 4
  artifacts:
    when: always
    paths:
      - MetaGuardReports/
    expire_in: 7 days
```

---

## Policy in CI

The CLI reads `metaguard_policy.json` from the project automatically. Ensure this file is committed to source control so the CI scan uses the same rules as developer machines.

To apply a stricter threshold on a release gate without modifying the policy file:

```bash
-- --format json --threshold 2
```

Issue class actions (`Block`, `Flag`, etc.) still apply on top of the threshold — a `Block` rule will fail the scan regardless of the threshold value.

---

## Caching in CI

The scan cache (`scan_cache.txt`) is based on file modification times. In CI environments where the workspace is cloned fresh on each run, the cache provides no benefit — MetaGuard detects a cold cache and runs a full scan automatically.

If your CI environment uses persistent workspaces or incremental checkouts, the cache will be effective between runs. In that case, include `Assets/MetaGuard/scan_cache.txt` in your cache restore key and add `MetaGuardReports/` to your artifact exclude list to avoid uploading stale reports.

---

## What the CLI Does Not Do

The CLI scans, analyzes, and reports. It does not apply fixes. Fix operations must be performed through the Editor. This is an intentional constraint — automated writes to project files in CI without a human review step and a rollback path would be unsafe.
