# Usage

---

## Opening the Window

```
Tools > MetaGuard > Open MetaGuard
```

The window docks like any standard Unity Editor panel and can be resized or repositioned freely.

---

## Pipeline Step Bar

The step bar at the top of the window tracks progress through the five pipeline stages:

```
Scan  ──►  Analyze  ──►  Simulate  ──►  Apply  ──►  Rollback
```

Each stage is activated after the previous one completes. Stages cannot be skipped.

---

## Toolbar

| Control | Default | Description |
|---|---|---|
| **Cache ON / Cache OFF** | ON | Toggles the scan cache. When ON, unchanged files are skipped on repeated scans. See [cache-system.md](cache-system.md). |

---

## Buttons

### Scan + Analyze

Starts a full project scan.

MetaGuard enumerates every asset in the `Assets/` folder, reads GUID data from `.meta` files, builds the dependency graph, and runs all issue detectors against the result. A progress indicator is shown during the scan. The operation can be cancelled.

**On completion:**
- Issues tab populates with all detected problems
- Project health score (0–100) is displayed
- Pipeline advances to Analyze

---

### Simulate

Tests every proposed fix against an in-memory copy of the dependency graph.

No files are written during this stage. The simulation examines downstream impact for each operation and assigns a verdict. Results appear in the **Results** tab when simulation completes.

**Simulation must complete before Apply is available.**

---

### Apply

Writes Safe and Warning operations to disk.

Before the first write, MetaGuard creates a complete snapshot of every affected file. A confirmation dialog is shown before writing begins. Dangerous and Destructive operations are not applied.

Apply is available only after a successful Simulate.

---

### Fix All Safe

Runs the complete pipeline — scan, simulate, and apply — in a single action. Only operations with a **Safe** verdict are written.

A confirmation dialog is shown before writing begins. The snapshot and rollback model is identical to the standard Apply flow.

**Use Fix All Safe for routine maintenance** on projects where the issue set is well understood and a fully automated pass is appropriate.

---

### Rollback

Restores all files modified during the most recent Apply session from the pre-apply snapshot.

The button is enabled whenever a valid snapshot session exists. After a successful rollback, the session is cleared and the button returns to a disabled state.

Rollback remains available for **48 hours** after Apply, including across editor restarts.

→ See [safety.md](safety.md) for the full rollback model.

---

## Tabs

### Issues

Displays all issues detected during the last scan, grouped by issue class and sorted by risk level.

| Control | Description |
|---|---|
| Search | Filter by asset name or path |
| Risk filter | Show only Critical, High, Medium, or Low issues |
| Type filter | Show only a specific issue class |

Click any issue row to expand the detail panel. The detail panel shows the affected asset path, issue class, risk level, and the proposed fix in plain language.

---

### Suggestions

Lists all fix suggestions generated from detected issues, ordered by priority.

Each entry shows:
- **Confidence** — How certain MetaGuard is that the fix is correct
- **Priority** — Relative ordering within the fix batch
- **Auto-eligible** — Whether this fix is included in Fix All Safe
- **Operation** — What the fix will do, in plain language

---

### Results

Populated after Simulate or Apply runs.

| Section | Content |
|---|---|
| Simulation verdicts | Safe / Warning / Dangerous / Destructive per operation |
| Apply write records | Files written, skipped, or failed during the last Apply |
| Auto-fix summary | Aggregate result when Fix All Safe was used |

---

### Log

Ring-buffered log of all pipeline activity. Use this to review exactly what MetaGuard did during a scan, simulate, or apply session.

| Control | Description |
|---|---|
| **Copy to Clipboard** | Copies the full log contents as plain text |
| **Clear** | Clears the log display |

The log holds up to 2,000 entries. Oldest entries are evicted automatically when the buffer is full.

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
