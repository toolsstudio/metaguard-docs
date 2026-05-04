> ⚠️ **This is an archived version (v1). Use [MetaGuard Pro (v2)](../../versions/v2/) documentation instead.**

---

# Getting Started

> First time using MetaGuard? This page takes you from installation to your first successful scan and rollback in under five minutes.

---

## 1. Install

MetaGuard 2.0.0 is available on the [Unity Asset Store](https://assetstore.unity.com/packages/slug/376206).

```
Assets > Import Package > Custom Package…
```

Select the downloaded `.unitypackage`, confirm all items are checked in the import dialog, and click **Import**.

MetaGuard installs to `Assets/MetaGuard/`. No additional setup is required.

→ See [installation.md](installation.md) for the full import reference, folder layout, and source control configuration.

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

## 3. Create a Policy File (First Use)

On first use, MetaGuard creates a default `metaguard_policy.json` in `Assets/MetaGuard/` automatically when the window opens.

To create it manually:

```
Tools > MetaGuard > Create Default Policy File
```

Commit `metaguard_policy.json` to source control so all team members and CI pipelines use the same rules.

→ See [policy.md](policy.md) for the full policy reference.

---

## 4. Run Your First Scan

Click **Scan + Analyze**.

MetaGuard enumerates every `.meta` file in the project, extracts GUID data, builds the asset dependency graph, and runs all issue detectors. A progress indicator is shown — the scan can be cancelled at any time without affecting the project.

When the scan completes:

- The **Issues** tab populates with all detected problems
- A project health score (0–100) appears at the top of the window
- The pipeline advances to Analyze

Wait for any pending Unity import (progress bar visible in the bottom-right of the Editor) to complete before scanning. Assets mid-import may have incomplete GUID data.

---

## 5. Review Detected Issues

Open the **Issues** tab.

Issues are grouped by class and sorted from Critical down to Low. Use the risk filter buttons or the search field to narrow the list. Click any issue row to expand its detail panel — it shows the affected asset path, issue class, risk level, GUID, reference count, and the proposed fix in plain language.

A score of 100 with an empty issues list means no problems were detected.

---

## 6. Simulate

Click **Simulate**.

Every proposed fix is tested against an in-memory copy of the dependency graph. No files are written at this stage. The simulation walks every downstream reference for each operation and assigns a verdict.

Open the **Results** tab when simulation completes. Each operation shows a verdict:

| Verdict | Meaning |
|---|---|
| **Safe** | Approved. No downstream references broken. |
| **Warning** | Approved with a noted caution. |
| **Dangerous** | Potential side effects — review before applying. |
| **Destructive** | Will break existing references — not applied automatically. |

Review the verdicts before proceeding. Dangerous and Destructive operations are never applied by Apply or Fix All Safe.

---

## 7. Apply

Click **Apply** to write Safe and Warning operations to disk.

A confirmation dialog appears first. Before any file is written, MetaGuard creates a complete snapshot of every affected asset — this snapshot is the source for rollback if you need to undo the changes.

Alternatively, click **Fix All Safe** to run the full pipeline — scan, simulate, and apply — in a single automated action. Fix All Safe writes only Safe verdicts. A confirmation dialog is shown before writing begins.

---

## 8. Verify

After Apply completes, run **Scan + Analyze** again. A project with no remaining issues should return a health score of 100. Check the **Results** tab for a record of everything that was written.

---

## 9. Rollback (If Needed)

If the result is not what you expected, click **Rollback**.

All files modified during the Apply session are restored from the pre-apply snapshot. Rollback is available for **48 hours** after Apply, including across editor restarts and domain reloads. After a successful rollback, the session is cleared and the button returns to a disabled state.

After rollback, run **Scan + Analyze** to confirm the project is back in its previous state.

---

## Next Steps

| Document | Description |
|---|---|
| [usage.md](usage.md) | Every button, tab, and control explained |
| [features.md](features.md) | Full feature reference |
| [policy.md](policy.md) | How per-project rules control issue handling and CI enforcement |
| [cli.md](cli.md) | Running MetaGuard in CI and batch mode |
| [safety.md](safety.md) | How the snapshot and rollback system works in detail |
| [cache-system.md](cache-system.md) | Faster repeated scans on large projects |
| [best-practices.md](best-practices.md) | Team workflows and source control recommendations |
| [troubleshooting.md](troubleshooting.md) | If something does not behave as expected |
