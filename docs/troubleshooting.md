# Troubleshooting

---

## Scan

### Scan returns 0 files

**Likely cause:** The `Assets/` folder cannot be located, or the project was recently moved or renamed.

**Resolution:**
1. Confirm the project has a valid `Assets/` folder in the project root.
2. Close and reopen the MetaGuard window, then scan again.
3. If the project was recently moved or renamed, restart the Unity Editor and retry.

---

### Scan completes but shows 0 issues on a project with known problems

**Likely cause:** The scan cache is skipping files whose modification timestamps were not updated when changes were made — for example, after a version control operation or an external tool that preserves timestamps.

**Resolution:**
1. Disable the cache using the **Cache ON / Cache OFF** toggle in the toolbar.
2. Run **Scan + Analyze** again.
3. If issues are now detected, the cache data was stale. Delete `MetaGuard/scan_cache.txt` to reset it, then re-enable the cache.

→ See [cache-system.md](cache-system.md) for more on when to disable the cache.

---

### MetaGuard's own assets appear in scan results

**Likely cause:** MetaGuard excludes its own assets from scan results automatically. If they appear, the project root path changed in a way that disrupted the self-exclusion filter.

**Resolution:** Restart the Unity Editor and run a fresh scan.

---

## Apply

### Apply aborts before writing any files

**Likely cause:** The pre-apply snapshot could not be completed. This occurs when a file is locked by another process, disk space is insufficient, or a file was deleted between scan and apply.

**Resolution:**
1. Ensure no other application has the affected files open.
2. Confirm there is sufficient disk space in the project directory.
3. Run **Scan + Analyze** to refresh the file list, then **Simulate** and **Apply** again.

---

### Some operations show "File Locked" after Apply

**Likely cause:** The target file was held open by Unity's import pipeline or another editor process at the time of the write.

**Resolution:**
1. Wait for the Unity import progress bar (bottom-right of the Editor) to complete.
2. Run **Apply** again — file-locked operations are retried automatically on re-apply.

---

### Apply completes but changes are not visible in the project

**Likely cause:** The Unity asset database has not refreshed yet.

**Resolution:** Select **Assets > Refresh** in the Unity menu bar, or press `Ctrl+R` / `Cmd+R`. If changes are still not visible, close and reopen the affected asset in the Inspector.

---

## Rollback

### The Rollback button is disabled

**Likely cause:** No active rollback session exists. This happens when no Apply has been run, when the previous rollback already completed and cleared the session, or when the session has expired (more than 48 hours since Apply).

**Resolution:** Rollback is available for 48 hours after each Apply, including across editor restarts. If the session has expired, restore the affected files from source control.

---

### Rollback reports "Snapshot Not Found"

**Likely cause:** The `MetaGuard/Snapshots/` directory has been deleted or moved — either manually or as a side effect of clearing the project's temporary folders.

**Resolution:**
1. Confirm `MetaGuard/Snapshots/` exists in the project root (not inside `Temp/`).
2. If the directory is missing, the snapshot is gone. Restore affected files from source control.

---

### Rollback reports a hash mismatch error

**Likely cause:** A snapshot file was modified after it was written. The most common cause is a version control tool normalizing line endings on files inside `MetaGuard/Snapshots/`.

**Resolution:**
1. Confirm `MetaGuard/Snapshots/` is in your `.gitignore`. MetaGuard includes this entry by default — verify it has not been removed.
2. Restore the affected files from source control.

---

## Cache

### Disabling the cache has no effect on scan results

**Likely cause:** The toggle was changed after the scan had already started, or the change was not saved before the scan began.

**Resolution:** Confirm **Cache OFF** is shown in the toolbar, then run **Scan + Analyze** again from the beginning.

---

### The cache does not reduce scan time on repeated runs

**Likely cause:** Cache data was not saved after the last scan, so no file records exist to skip.

**Resolution:** Run **Scan + Analyze** with Cache ON enabled. If the problem persists, delete `MetaGuard/scan_cache.txt` and run a full scan — MetaGuard will rebuild the cache file from scratch.

---

## UI

### The Log tab shows no entries

**Likely cause:** No pipeline operations have been run since the window was opened, or the log was cleared manually.

**Resolution:** Run **Scan + Analyze**. The log is written to as soon as the scan starts.

---

### The MetaGuard menu item does not appear

**Likely cause:** The package was not imported correctly, or compilation errors are preventing editor scripts from loading.

**Resolution:**
1. Open the Unity Console and look for compilation errors.
2. Re-import the MetaGuard `.unitypackage` via **Assets > Import Package > Custom Package…**
3. If errors persist, confirm the project meets the minimum Unity version requirement (2020.3 LTS or later).
