# Usage

Complete reference for every control, tab, and button in the MetaGuard window.

---

## Opening the Window

```
Tools > MetaGuard > Open MetaGuard
```

The window docks like any standard Unity Editor panel and can be resized or repositioned freely.

---

## Header

The header bar displays:
- Current version (`Version 2.0.0`)
- Scan mode indicator (`Watching` when the file watcher is active)
- Project health score (updated after each completed scan)
- `Dirty (N)` badge — appears when assets have changed since the last scan
- Cache state toggle (`Cache ON` / `Cache OFF`)

---

## Pipeline Step Bar

The step bar at the top of the window tracks progress through the five pipeline stages:

```
Scan  ──►  Analyze  ──►  Simulate  ──►  Apply  ──►  Rollback
```

Each stage activates after the previous one completes. A checkmark (✓) marks completed stages. Stages cannot be skipped.

---

## Toolbar

| Control | Default | Description |
|---|---|---|
| **Cache ON / Cache OFF** | ON | Toggles the scan cache. When ON, unchanged files are skipped on repeated scans. See [cache-system.md](cache-system.md). |

---

## Primary Buttons

### Scan + Analyze (Scan Project)

Starts a full project scan.

MetaGuard enumerates every asset in the project (Assets and Packages by default), reads GUID data from `.meta` files, builds the dependency graph, and runs all seven issue detectors. The scan runs on a background thread — the Editor remains responsive throughout. A progress indicator is shown during the scan; click it to cancel.

**On completion:**
- Issues tab populates with all detected problems
- Project health score (0–100) is displayed
- Pipeline advances to Analyze

---

### Simulate

Tests every proposed fix against an in-memory copy of the dependency graph.

No files are written during this stage. The simulation walks downstream references for each operation and assigns a verdict. Results appear in the **Results** tab when simulation completes.

**Simulation must complete before Apply is available.**

---

### Apply (Apply Fixes)

Writes Safe and Warning operations to disk.

Before the first write, MetaGuard creates a complete snapshot of every affected file. A confirmation dialog is shown before writing begins. Dangerous and Destructive operations are not applied.

Apply is available only after a successful Simulate.

---

### Fix All Safe

Runs the complete pipeline — scan, simulate, and apply — in a single automated action. Only operations with a **Safe** verdict are written.

A confirmation dialog is shown before writing begins. The snapshot and rollback model is identical to the standard Apply flow.

Use Fix All Safe for routine maintenance on projects where the issue set is well understood and a conservative, fully automated pass is appropriate.

---

### Preview Changes

Opens a read-only view of all operations queued for the next Apply, including their verdicts and affected file paths. Available after Simulate completes.

---

### Rollback

Restores all files modified during the most recent Apply session from the pre-apply snapshot.

The button is enabled whenever a valid snapshot session exists. After a successful rollback, the session is cleared and the button returns to a disabled state.

Rollback remains available for **48 hours** after Apply, including across editor restarts and domain reloads.

→ See [safety.md](safety.md) for the full rollback model.

---

### Allow Warnings

When enabled, operations with a Warning verdict are included in Apply and Fix All Safe. When disabled, only Safe operations are written. Toggle state persists across sessions.

---

### Fix Selected

Applies only the operations currently selected in the Suggestions tab. Requires a completed Simulate. A snapshot is created identically to a full Apply.

---

### Policy

Opens the Policy tab directly.

---

### Clear Log

Clears all entries from the Log tab. Does not affect scan state or session data.

---

## Tabs

### Issues

Displays all issues detected during the last scan, grouped by class and sorted by risk level (Critical first).

| Control | Description |
|---|---|
| Search | Filter by asset name or path (case-insensitive substring match) |
| All / Coll / Zero / Malf / Orph / Miss / BrkRef / Cycle | Type filter — show only a specific issue class |
| Crit+High Only | Show only Critical and High issues |
| Risk filter | Show only Critical, High, Medium, or Low issues |
| Issue row (click to expand) | Shows asset path, issue class, risk level, GUID, reference count, and proposed fix |
| Delete button (row) | Dismisses the issue from the current session (does not affect the project file) |
| `+` button (row) | Adds the issue to the Fix Selected queue |

---

### Suggestions

Lists all fix suggestions generated from detected issues, ordered by priority.

Each entry shows:
- **Confidence** — How certain MetaGuard is that the fix is correct (Certain / High / Medium / Low)
- **Priority** — Relative ordering within the fix batch
- **Auto-eligible** — Whether this fix is included in Fix All Safe and Fix Selected
- **Operation** — What the fix will do, in plain language

---

### Results

Populated after Simulate or Apply runs.

**After Simulate:** Each queued operation is shown with its verdict, the number of affected asset graph nodes, and the estimated change in issue count.

**After Apply:** Each operation that was written, skipped, or failed is shown, with timing and the final write path.

**After Fix All Safe:** Shows the aggregate auto-fix summary — how many operations were generated, simulated, and applied, and how many files were written.

---

### Log

Ring-buffered log of all pipeline activity. Entries are tagged with timestamp, stage, and severity.

| Control | Description |
|---|---|
| **Copy** | Copies the full log contents as plain text |
| **Clear** | Clears the log display |

The buffer holds up to 2,000 entries. Oldest entries are evicted automatically when the buffer is full. The log is not persisted to disk — it is cleared when the window is closed.

---

### Policy

Displays the currently loaded policy configuration.

| Control | Description |
|---|---|
| Reload | Re-reads `metaguard_policy.json` from disk |
| Create Default File | Creates the default policy file if one does not exist |
| Rules table | Shows each issue class, its current action, and a description of what the action means |
| Action legend | AutoFix / Flag / Ignore / Block explained inline |

The notice at the bottom of the Policy tab confirms the loaded file path and reminds you that the same file is used by the CLI.

→ See [policy.md](policy.md) for the full policy reference.

---

### History

Shows the health score and issue counts from every completed scan in the current project. Scores are recorded automatically to `MetaGuard/health_log.json` after each scan.

Commit `health_log.json` to source control to track project health over time and share history across the team.

---

### About

Displays version, publisher, and support links.

| Button | Destination |
|---|---|
| Documentation | https://github.com/toolsstudio/metaguard-docs |
| Rate on Asset Store | Asset Store product page |
| Discord Support | https://discord.gg/rYbZZz5GH4 |
| Email Support | tools.studio@zohomail.in |
| Report a Bug (Discord) | https://discord.gg/mQYguyhYwA |

---

## Recommended Workflow

```
1.  Open MetaGuard
2.  Click Scan + Analyze
3.  Review Issues tab — note Critical and High items
4.  Click Simulate
5.  Review Results tab — confirm intended operations show Safe
6.  Click Apply  (or Fix All Safe for a fully automated pass)
7.  Verify the project — run a follow-up scan to confirm health score
8.  If the result is unexpected — click Rollback
```
