# Features

---

## Full Project Scan

MetaGuard enumerates every `.meta` file in the `Assets/` folder, extracts the GUID assigned to each asset, and maps the relationship between each asset and its corresponding `.meta` file.

The scan runs in the background — the Unity Editor remains responsive throughout. A progress indicator is shown, and the scan can be cancelled at any time without affecting the project.

---

## Dependency Graph

After scanning, MetaGuard constructs a directed reference graph from asset data. The graph captures which assets reference which other assets, and forms the foundation for simulation — without it, MetaGuard cannot determine whether a proposed fix will break downstream references before the fix is applied.

---

## Seven-Class Issue Detection

| Issue Class | Description |
|---|---|
| **GUID Collision** | Two or more assets share the same GUID |
| **Zero GUID** | A `.meta` file contains an all-zero GUID value |
| **Malformed GUID** | A GUID entry does not conform to the expected format |
| **Orphaned `.meta`** | A `.meta` file exists with no corresponding asset |
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

A health score from 0 to 100 is reported after each scan. A score of 100 indicates no detected issues. The score reflects the aggregate severity of all detected issues, weighted by risk level.

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

Apply writes only Safe and Warning operations. Each write is staged through a temporary file — if the write is interrupted by a crash or power loss, the original file is not corrupted.

All affected assets are refreshed in the Unity asset database after all writes complete. MetaGuard does not trigger per-file reimports during the write pass.

---

## Auto-Fix Controller

The auto-fix pipeline handles the most common, low-risk issue classes automatically:

- **GUID regeneration** — Assigns a new unique GUID to assets with zero or colliding GUIDs, and updates all references.
- **Orphan deletion** — Removes `.meta` files that have no corresponding asset.
- **Missing meta creation** — Generates a new `.meta` file for assets that are missing one.

**Fix All Safe** runs the full auto-fix pipeline — scan, simulate, and apply — in a single action, applying only Safe verdicts.

---

## Scan Cache

The scan cache tracks file modification times. On repeated scans, files that have not changed since the last scan are skipped. On large projects with stable asset sets, this significantly reduces scan time after the first run.

The cache is enabled by default and can be toggled in the MetaGuard toolbar. Cache data is stored in `MetaGuard/scan_cache.txt`.

→ See [cache-system.md](cache-system.md) for details on when to disable or reset the cache.

---

## Three-Tier Rollback

MetaGuard enforces three layers of protection on every Apply operation:

| Tier | Description |
|---|---|
| **Pre-apply snapshot** | All affected files copied to an archive before any write begins |
| **Persistent session** | Snapshot survives editor restarts, domain reloads, and crashes. Available for 48 hours. |
| **Automatic failure rollback** | Any exception during Apply triggers automatic restore — partial state is not possible |

→ See [safety.md](safety.md) for the complete model.

---

## Limitations

- MetaGuard reads `.meta` files and YAML-serialized asset data. Binary asset formats — textures, meshes, audio — are not parsed for embedded GUID references.
- Scene (`.unity`) and prefab (`.prefab`) files are not modified directly. Reference repair operates on `.meta` files and YAML asset data.
- Snapshot archives are not version-controlled and are not a substitute for source control.
- Rollback sessions expire after 48 hours.
- YAML reference extraction covers standard Unity serialization. Non-standard serializers may produce false negatives in reference analysis.
