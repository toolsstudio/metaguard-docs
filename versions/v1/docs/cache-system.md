> ⚠️ **This is an archived version (v1). Use [MetaGuard Pro (v2)](../../versions/v2/) documentation instead.**

---

# Cache System

---

## What the Cache Does

The scan cache tracks the last-modified time of each asset file in the project. On repeated scans, files that have not changed since the previous scan are skipped — only new and modified files are re-read from disk.

For projects with large, stable asset directories, the cache reduces scan time significantly after the first full scan. The first scan after enabling the cache is always a full scan; subsequent scans benefit from the cached data.

---

## Enabling and Disabling

The cache is controlled by the **Cache ON / Cache OFF** toggle in the MetaGuard toolbar.

| State | Behavior |
|---|---|
| **Cache ON** (default) | Unchanged files are skipped on scan |
| **Cache OFF** | Every file is read on every scan, regardless of modification time |

The toggle takes effect on the next scan. Changing it while a scan is in progress has no effect on the current operation.

---

## When to Keep the Cache Enabled

Cache ON is the right choice for most day-to-day workflows, particularly when:

- The project has hundreds or thousands of assets
- Scans are run frequently as part of a regular workflow
- Most assets are stable between scan sessions

---

## When to Disable the Cache

Disable the cache when you need a guaranteed full scan of every file. Common scenarios:

- The previous scan returned unexpected results — for example, zero issues on a project with known problems
- Files were modified outside of Unity — by a version control operation, external script, or filesystem tool that did not update modification timestamps
- Assets were moved or renamed in bulk and scan results appear inconsistent with expected state
- You are running a validation scan and need to confirm the result is not influenced by stale cached data

After a clean full scan with Cache OFF, re-enable the cache for subsequent scans.

---

## Resetting the Cache

Cache data is stored in `MetaGuard/scan_cache.txt`.

To fully reset the cache, delete this file. MetaGuard recreates it automatically on the next scan with Cache ON active.

Deleting `scan_cache.txt` is safe at any time and has no effect on project assets or rollback sessions.

---

## Cache Limitations

- The cache tracks modification time, not file contents. If a file's modification time is unchanged but its contents were altered by an external tool that preserves timestamps, the cache will not detect the change. Disable the cache and run a full scan if this is a concern.
- Cache data is local to the machine. It is not shared across team members and should not be committed to version control.
- Cache state resets if `scan_cache.txt` is deleted, moved, or excluded from the project.

---

## Source Control

Add `scan_cache.txt` to your `.gitignore`:

```
Assets/MetaGuard/scan_cache.txt
```

MetaGuard includes this entry by default. Each team member builds their own cache from their first local scan.

---

## Cache in CI

The scan cache is based on filesystem modification times. In CI environments where the workspace is a fresh clone on every run, the cache will always be cold — every scan will be a full scan. This is the correct behaviour and requires no configuration.

If your CI environment uses persistent workspaces or incremental checkouts, the cache will be effective between runs on the same workspace. Include `Assets/MetaGuard/scan_cache.txt` in your workspace cache key to preserve it across runs, and ensure it is invalidated when a full re-scan is required (for example, on a new branch or after a large merge).
