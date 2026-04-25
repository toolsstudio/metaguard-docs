# Safety

> MetaGuard enforces three layers of protection on every Apply operation. No file is modified without a recoverable snapshot. No operation reaches Apply without passing simulation.

---

## The Three-Tier Model

### Tier 1 — Pre-Apply Snapshot

Before any project file is modified, MetaGuard writes a complete snapshot of every file that Apply will touch.

- If snapshot creation fails for any file, Apply aborts before writing a single project file.
- The snapshot is stored in `MetaGuard/Snapshots/` in the project root.
- Each snapshot is tied to a unique session identifier.

The snapshot is written once, at the start of Apply, and is never modified afterward.

---

### Tier 2 — Persistent Session Archive

The snapshot session persists across editor restarts, domain reloads, and crashes.

- Rollback is available immediately after Apply completes.
- Rollback remains available for **48 hours** — even after closing and reopening the Unity Editor.
- The session record is stored durably and survives unexpected shutdowns.

After 48 hours, the session is considered expired. Rollback for that session is no longer available.

---

### Tier 3 — Automatic Failure Rollback

Any exception during Apply triggers an automatic restore from the snapshot before the error is surfaced.

- The project is never left in a partial state after a failed Apply.
- Each file write is staged through a temporary path and moved into place only after the write completes. If the move fails, the original file is untouched.
- After a successful automatic rollback, the session is cleared.

---

## Snapshot Integrity Validation

Before any restore, MetaGuard validates each snapshot file against a stored hash. If a snapshot file has been altered after it was written, Rollback aborts with a diagnostic error rather than restore potentially incorrect data.

A corrupt snapshot cannot be restored by MetaGuard. If a snapshot is corrupt, restore the affected files from source control.

---

## Simulation Gate

No operation reaches Apply without a simulation verdict. Every proposed fix is tested against an in-memory copy of the dependency graph before Apply is permitted to run.

- **Dangerous** and **Destructive** operations are not applied by Apply or Fix All Safe.
- The simulation writes nothing — no files are touched during this stage.

---

## How to Roll Back

Click **Rollback** in the MetaGuard window.

The button is enabled whenever a valid snapshot session exists. After a successful rollback, the session is cleared and the button returns to a disabled state.

**Rollback does not re-run a scan.** Run **Scan + Analyze** after rolling back to confirm the project is in the expected state.

---

## What MetaGuard Does Not Protect Against

| Scenario | Recommendation |
|---|---|
| Changes made more than 48 hours after Apply | Restore from source control |
| Snapshot files corrupted by external tools (e.g. line-ending normalization) | Add `MetaGuard/Snapshots/` to `.gitignore` |
| Manual edits made to affected files after Apply | Rollback overwrites post-Apply manual changes — ensure this is intentional |
| Absence of source control | MetaGuard's snapshot is a safety net, not a substitute for version control |

---

## Recommended Practices

- Commit to source control before running Apply on significant issue sets.
- Confirm `MetaGuard/Snapshots/` is in your `.gitignore`. MetaGuard includes this entry by default.
- Run **Scan + Analyze** after rollback to verify project state.
- Do not delete `MetaGuard/Snapshots/` while a rollback session may still be needed.
