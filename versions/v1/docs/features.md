> ⚠️ **This is an archived version (v1). Use [MetaGuard Pro (v2)](../../versions/v2/) documentation instead.**

---

# Features

---

## Full Project Scan

MetaGuard enumerates every `.meta` file in the project, extracts the GUID assigned to each asset, and maps the relationship between each asset and its corresponding `.meta` file.

The default scan depth is **AssetsAndPackages** — both the `Assets/` folder and the `Packages/` folder are indexed. This is required to correctly resolve GUID references made by pipeline-managed assets (such as URP materials that reference shader GUIDs inside `Packages/com.unity.render-pipelines.universal/`). Without package indexing, these references appear broken even though they are valid at runtime.

The scan runs in the background — the Unity Editor remains responsive throughout. A progress indicator is shown, and the scan can be cancelled at any time without affecting the project.

---

## Dependency Graph

After scanning, MetaGuard constructs a directed reference graph from asset YAML serialization data. The graph captures which assets reference which other assets by GUID, forming the foundation for the simulation stage. Without the graph, MetaGuard cannot determine whether a proposed fix will break downstream references before applying it.

---

## Seven-Class Issue Detection

| Issue Class | Description |
|---|---|
| **GUID Collision** | Two or more assets share the same GUID |
| **Zero GUID** | A `.meta` file contains an all-zero GUID value (`000…000`) |
| **Malformed GUID** | A GUID entry does not conform to the expected 32-hex-character format |
| **Orphaned `.meta`** | A `.meta` file exists with no corresponding asset on disk |
| **Missing `.meta`** | An asset file exists with no corresponding `.meta` file |
| **Broken Reference** | An asset references a GUID that cannot be resolved in the project |
| **Cyclic Dependency** | A group of assets form a circular reference chain |

MetaGuard's own files are excluded from scan results automatically.

---

## Risk Classification

Every detected issue is assigned one of four risk levels:

| Level | Meaning |
|---|---|
| **Critical** | Immediate impact on project integrity or build correctness |
| **High** | Significant risk; likely to cause reference failures |
| **Medium** | Moderate risk; may surface during import or build |
| **Low** | Minor; no immediate functional impact |

Risk is determined by downstream reference count and fix complexity. Critical and High issues appear first in the Issues tab.

---

## Project Health Score

A health score from 0 to 100 is reported after each scan. A score of 100 indicates no detected issues. The score reflects the aggregate severity of all detected issues, weighted by risk level and filtered by the current policy (issues set to Ignore do not affect the score).

The score is recorded to `MetaGuard/health_log.json` after each scan. Commit this file to track project health over time.

---

## Policy System

A per-project `metaguard_policy.json` file controls how each issue class is handled in the Editor, during Fix All Safe, and in CLI scans. Committing the file to source control ensures consistent enforcement across team members and CI pipelines.

Four actions are available per issue class:

| Action | Behavior |
|---|---|
| **AutoFix** | Fix suggestion generated and marked auto-eligible. Applied by Fix All Safe and CLI without confirmation. |
| **Flag** | Fix suggestion generated, surfaced in Suggestions tab, but not applied automatically. User must apply explicitly. |
| **Ignore** | Issue detected but not surfaced, not included in health score, not reported in CLI output. |
| **Block** | In Editor: treated as Flag. In CLI: forces `has_violations = true` and exit code 1 regardless of other counts. |

The `excludeAssetPaths` field in the policy file lists project-relative paths excluded from BrokenReference analysis. This is used to suppress false positives from pipeline-managed assets (URP global settings, default volume profiles) without disabling BrokenReference detection globally.

→ See [policy.md](policy.md) for the complete policy reference.

---

## Simulation Engine

Every proposed fix is tested against an in-memory copy of the dependency graph before any file is touched. The simulation walks downstream references for each operation and assigns a verdict:

| Verdict | Meaning |
|---|---|
| **Safe** | Approved. No references broken. |
| **Warning** | Approved with a noted caution. |
| **Dangerous** | Potential side effects — review before applying. |
| **Destructive** | Will break existing references — not applied automatically. |

The simulation produces no side effects. No files are written or modified during this stage.

---

## Safe Apply Pipeline

Apply writes only Safe and Warning operations. Each write is staged through a temporary file — if the write is interrupted by a crash or power loss, the original file is not corrupted. All affected assets are refreshed in the Unity asset database after all writes complete. MetaGuard does not trigger per-file reimports during the write pass.

---

## Auto-Fix Controller

The auto-fix pipeline handles the most common, low-risk issue classes automatically:

- **GUID regeneration** — Assigns a new unique GUID to assets with zero or colliding GUIDs, and updates all known references.
- **Orphan deletion** — Removes `.meta` files that have no corresponding asset on disk.
- **Missing meta creation** — Generates a new `.meta` file for assets that are missing one.

**Fix All Safe** runs the full auto-fix pipeline — scan, simulate, and apply — in a single action, applying only Safe verdicts.

---

## Scan Cache

The scan cache tracks file modification times. On repeated scans, files that have not changed since the last scan are skipped. On large projects with stable asset sets, this significantly reduces scan time after the first run.

The cache is enabled by default and can be toggled in the MetaGuard toolbar. Cache data is stored in `MetaGuard/scan_cache.txt` and should not be committed to source control.

→ See [cache-system.md](cache-system.md) for details on when to disable or reset the cache.

---

## CLI / CI Integration

MetaGuard exposes a headless CLI entry point for Unity batch mode:

```bash
Unity -batchmode \
  -projectPath /path/to/project \
  -executeMethod ToolsStudio.MetaGuard.Editor.CLI.MetaGuardCLI.Run \
  -logFile metaguard.log -quit \
  -- --format json --threshold 4
```

Reports are written automatically to `MetaGuardReports/` when `--output` is not specified. Exit codes are deterministic: `0` = clean, `1` = violations, `2` = scan failure.

→ See [cli.md](cli.md) for the full CLI reference, including GitHub Actions and GitLab CI examples.

---

## Health History

MetaGuard records the health score and issue counts from every completed scan to `MetaGuard/health_log.json`. The History tab displays this data as a timeline. Commit the file to source control to share health history across the team and track trends over time.

---

## Three-Tier Rollback

MetaGuard enforces three layers of protection on every Apply operation:

| Tier | Description |
|---|---|
| **Pre-apply snapshot** | All affected files copied to an archive before any write begins |
| **Persistent session** | Snapshot survives editor restarts, domain reloads, and crashes. Available for 48 hours. |
| **Automatic failure rollback** | Any exception during Apply triggers automatic restore — partial project state is not possible |

→ See [safety.md](safety.md) for the complete model.

---

## Limitations

- MetaGuard reads `.meta` files and YAML-serialized asset data. Binary asset formats — textures, meshes, audio — are not parsed for embedded GUID references.
- Scene (`.unity`) and prefab (`.prefab`) files are not modified directly. Reference repair operates on `.meta` files and YAML asset data.
- Snapshot archives are not version-controlled and are not a substitute for source control.
- Rollback sessions expire after 48 hours.
- YAML reference extraction covers standard Unity serialization. Non-standard serializers may produce false negatives in reference analysis.
- The CLI scans and reports only — it does not apply fixes.
