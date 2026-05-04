> ⚠️ **This is an archived version (v1). Use [MetaGuard Pro (v2)](../../versions/v2/) documentation instead.**

---

# Safety & Rollback

> MetaGuard enforces three layers of protection on every Apply operation. No file is modified without a recoverable snapshot. No operation reaches Apply without passing simulation.

---

## The Three-Tier Model

### Tier 1 — Pre-Apply Snapshot

Before any project file is modified, MetaGuard writes a complete snapshot of every file that Apply will touch.

Each snapshot entry contains:
- The original file contents (verbatim binary copy)
- A SHA-256 hash of the original file
- The project-relative path

The snapshot is stored in `MetaGuard/Snapshots/<session_id>/`. Each session has a unique ID in the format `YYYYMMDD_HHmmss_<suffix>`.

If snapshot creation fails for any file, Apply aborts before writing a single project file. The project is never modified if the snapshot phase does not complete successfully.

The snapshot is written once, at the start of Apply, and is never modified afterward.

---

### Tier 2 — Persistent Session Archive

The snapshot session persists across editor restarts, domain reloads, and crashes.

- Rollback is available immediately after Apply completes.
- Rollback remains available for **48 hours** — even after closing and reopening the Unity Editor.
- The session record is stored durably on disk and survives unexpected shutdowns.

After 48 hours, the session is considered expired. Rollback for that session is no longer available, and the session data is pruned automatically.

---

### Tier 3 — Automatic Failure Rollback

Any exception during Apply triggers an automatic restore from the snapshot before the error is surfaced.

- The project is never left in a partial state after a failed Apply.
- Each file write is staged through a temporary path and moved into place atomically only after the write completes. If the move fails, the original file is untouched.
- After a successful automatic rollback, the session is cleared.

---

## Snapshot Integrity Validation

Before any restore — whether triggered by the Rollback button or by an automatic failure — MetaGuard validates each snapshot file against its stored SHA-256 hash.

If a snapshot file has been altered after it was written (for example, by a version control tool normalizing line endings), the hash will not match. Rollback aborts for that file and logs a diagnostic error. The remaining files in the session are still restored.

A file that cannot be restored due to a hash mismatch or missing snapshot entry must be recovered from source control.

---

## Simulation Gate

No operation reaches Apply without a simulation verdict. Every proposed fix is tested against an in-memory copy of the dependency graph before Apply is permitted to run.

- **Dangerous** and **Destructive** operations are never applied by Apply or Fix All Safe.
- The simulation writes nothing — no files are touched during this stage.
- If the simulation has not run since the last scan, Apply is not available.

---

## Apply Write Model

Each file write in Apply follows this sequence:

1. Write the new content to a temporary file alongside the target
2. Verify the write completed without error
3. Replace the original with the temp file atomically
4. Record the write in the session log

If any step fails, the exception triggers automatic rollback of all writes made so far in the current session. A partial Apply is not possible.

---

## Session Lifecycle

A session exists from the moment Apply begins until rollback is completed or the 48-hour expiry elapses.

Sessions survive:
- Domain reloads
- Editor restarts
- Unity crashes (the session data is on disk before any project file is touched)

On editor startup, MetaGuard checks for sessions created in the last 48 hours and makes the Rollback button available if one is found. The session log records each write as it completes — if a crash occurred mid-Apply, the log identifies exactly which files were written before the crash.

---

## How to Roll Back

Click **Rollback** in the MetaGuard window.

The button is enabled whenever a valid snapshot session exists. After a successful rollback, the session is cleared and the button returns to a disabled state.

**Rollback does not re-run a scan.** Run **Scan + Analyze** after rolling back to confirm the project is in the expected state.

---

## What MetaGuard Does Not Modify

MetaGuard's Apply engine only modifies `.meta` files and GUID fields within YAML asset data. It does not:

- Modify binary asset file contents (textures, meshes, audio, compiled scripts)
- Modify `ProjectSettings/`
- Modify files outside the `Assets/` directory
- Modify `Packages/`

Simulation verdicts are computed against these constraints. An operation that would require modifying a binary asset rather than its `.meta` receives a Dangerous or Destructive verdict and is not applied automatically.

---

## Crash Recovery

If Unity crashes during an Apply session:

1. Reopen the project.
2. Open MetaGuard via `Tools > MetaGuard > Open MetaGuard`.
3. The session initializer detects the interrupted session at startup and activates the Rollback button.
4. Click **Rollback** to restore the pre-apply state.

Files that were fully written before the crash are included in rollback. Files that were mid-write may be in an indeterminate state — `AssetDatabase.Refresh` will detect and report this on next import.

---

## What MetaGuard Does Not Protect Against

| Scenario | Recommendation |
|---|---|
| Changes made more than 48 hours after Apply | Restore from source control |
| Snapshot files corrupted by external tools (e.g. line-ending normalization) | Add `MetaGuard/Snapshots/` to `.gitignore` |
| Manual edits made to affected files after Apply | Rollback overwrites post-Apply manual changes — confirm this is intentional before rolling back |
| Absence of source control | MetaGuard's snapshot is a safety net, not a substitute for version control |

---

## Recommended Practices

- Commit to source control before running Apply on a significant issue set.
- Confirm `MetaGuard/Snapshots/` is in your `.gitignore`. MetaGuard includes this entry by default — verify it has not been removed.
- Run **Scan + Analyze** after rollback to verify the project returned to its expected state.
- Do not delete `MetaGuard/Snapshots/` while a rollback session may still be needed.
