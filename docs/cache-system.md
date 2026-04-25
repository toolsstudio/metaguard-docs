# Cache System

---

## What the Cache Does

The scan cache tracks the last-modified time of each asset file in the project. On repeated scans, files that have not changed since the previous scan are skipped — only new and modified files are re-read.

For projects with large, stable asset directories, the cache reduces scan time significantly after the first full scan.

---

## Enabling and Disabling

The cache is controlled by the **Cache ON / Cache OFF** toggle in the MetaGuard toolbar.

| State | Behavior |
|---|---|
| **Cache ON** (default) | Unchanged files are skipped on scan |
| **Cache OFF** | Every file is read on every scan, regardless of modification time |

The toggle takes effect on the next scan. Changing it mid-scan has no effect on the current operation.

---

## When to Keep the Cache Enabled

Cache ON is the right choice for most day-to-day workflows, particularly when:

- The project has hundreds or thousands of assets
- Scans are run frequently as part of a regular workflow
- Most assets are stable between scan sessions

---

## When to Disable the Cache

Disable the cache when you need a guaranteed full scan of every file. Common scenarios:

- The previous scan returned unexpected results — zero issues on a project with known problems
- Files were modified outside of Unity — by a version control operation, external script, or file system tool that did not update modification timestamps
- Assets were moved or renamed in bulk and scan results appear inconsistent

After a clean full scan, re-enable the cache for subsequent scans.

---

## Resetting the Cache

Cache data is stored in `MetaGuard/scan_cache.txt`.

To fully reset the cache, delete this file. MetaGuard recreates it automatically on the next scan with Cache ON active.

Deleting `scan_cache.txt` is safe at any time and has no effect on project assets or snapshot sessions.

---

## Cache Limitations

- The cache tracks modification time, not file contents. If a file's modification time is unchanged but its contents were altered by an external tool that preserves timestamps, the cache will not detect the change. Disable the cache and run a full scan if this is a concern.
- Cache data is local to the machine. It is not shared across team members and should not be committed to version control.
- Cache state resets if `scan_cache.txt` is deleted, moved, or excluded from the project.
