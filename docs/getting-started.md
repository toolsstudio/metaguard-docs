# Getting Started

> First time using MetaGuard? This page takes you from installation to your first successful scan and rollback in under five minutes.

---

## 1. Install

Download and import the MetaGuard `.unitypackage` into your Unity project. MetaGuard is free — available on [GitHub](https://github.com/Afterix-Hub/MetaGuard), the [Unity Asset Store](https://assetstore.unity.com/packages/slug/376206), [itch.io](https://tools-studio.itch.io/MetaGuard), and [Gumroad](https://toolsstudio.gumroad.com/l/MetaGuard).

```
Assets > Import Package > Custom Package…
```

Select the downloaded file, confirm all items are checked in the import dialog, and click **Import**.

MetaGuard installs to `Assets/MetaGuard/`. No additional setup is required.

→ See [installation.md](installation.md) for the full import reference.

---

## 2. Open MetaGuard

```
Tools > MetaGuard > Open MetaGuard
```

The MetaGuard window opens and docks like any standard Unity Editor panel. At the top of the window, the pipeline step bar tracks where you are:

```
Scan  ──►  Analyze  ──►  Simulate  ──►  Apply  ──►  Rollback
```

Stages activate as you move through the pipeline. You cannot skip a stage.

---

## 3. Run Your First Scan

Click **Scan + Analyze**.

MetaGuard enumerates every `.meta` file in the project, extracts GUID data, builds the asset dependency graph, and runs all issue detectors. A progress indicator is shown — the scan can be cancelled at any time.

When the scan completes:

- The **Issues** tab populates with all detected problems
- A project health score (0–100) appears at the top of the window
- The pipeline advances to **Analyze**

---

## 4. Review Detected Issues

Open the **Issues** tab.

Issues are grouped by class and sorted from Critical down to Low. Use the risk filter or type filter to narrow the list. Click any issue row to expand its detail panel — it shows the affected asset path, issue type, risk level, and the proposed fix.

A score of 100 with an empty issues list means no problems were detected.

---

## 5. Simulate

Click **Simulate**.

Every proposed fix is tested against an in-memory copy of the dependency graph. No files are written at this stage.

Open the **Results** tab when simulation completes. Each operation shows a verdict:

| Verdict | Meaning |
|---|---|
| **Safe** | Approved. No downstream references broken. |
| **Warning** | Approved with a noted caution. |
| **Dangerous** | Potential side effects — review before applying. |
| **Destructive** | Will break references — not applied automatically. |

Review the results before proceeding.

---

## 6. Apply

Click **Apply** to write Safe and Warning operations to disk.

A confirmation dialog appears first. Before any file is written, MetaGuard creates a complete snapshot of all affected assets. The snapshot is stored in `MetaGuard/Snapshots/` and is the source for rollback.

Alternatively, click **Fix All Safe** to apply only Safe operations in a single automated action — scan, simulate, and apply in one step.

---

## 7. Verify

After Apply completes, check the Unity Console and inspect the affected assets to confirm the expected result. Run **Scan + Analyze** again — a clean project should return a health score of 100.

---

## 8. Rollback (If Needed)

If the result of Apply is not what you expected, click **Rollback**.

All files modified during the Apply session are restored from the pre-apply snapshot. Rollback is available for **48 hours** after Apply, including across editor restarts.

After rollback, run **Scan + Analyze** to confirm the project is back in its previous state.

---

## Next Steps

| Document | Description |
|---|---|
| [usage.md](usage.md) | Every button, tab, and control explained |
| [features.md](features.md) | Full feature reference |
| [safety.md](safety.md) | How the snapshot and rollback system works |
| [cache-system.md](cache-system.md) | Faster repeated scans on large projects |
| [troubleshooting.md](troubleshooting.md) | If something does not behave as expected |
