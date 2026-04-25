# FAQ

---

## General

**What does MetaGuard actually change in my project?**

MetaGuard reads `.meta` files and YAML asset serialization data to detect GUID integrity issues. When Apply runs, it writes corrected content to affected `.meta` and asset files. It does not modify binary assets, scene files, or prefab files directly. Every change is recorded in a snapshot before it is made.

---

**Is it safe to run on a production project?**

Yes. MetaGuard creates a complete snapshot of all affected files before writing anything. If the result is not what you expected, Rollback restores the previous state in a single click — and remains available for 48 hours, including after restarting the Editor.

That said: committing to source control before running Apply on a significant issue set is always the right move. The snapshot system is a safety net — it does not replace version control.

---

**Does MetaGuard add anything to my build?**

No. All MetaGuard scripts are in Editor-only assemblies and are excluded from all builds automatically. There is no runtime footprint.

---

**Does MetaGuard modify scene or prefab files?**

No. MetaGuard repairs GUID fields in `.meta` files and YAML asset data. Scene (`.unity`) and prefab (`.prefab`) files are not modified.

---

**Does MetaGuard work with all render pipelines?**

Yes. MetaGuard has no render pipeline dependency. It operates on `.meta` files and Unity's YAML asset serialization, which is pipeline-agnostic.

---

## Scanning

**How long does a scan take?**

On a project with a few hundred assets, a full scan completes in seconds. On very large projects, enable the scan cache — after the first full scan, subsequent scans skip unchanged files and complete much faster.

→ See [cache-system.md](cache-system.md).

---

**Should I wait for Unity to finish importing before scanning?**

Yes. Assets mid-import may have incomplete GUID data. Wait for the import progress bar to clear before starting a scan.

---

**Does MetaGuard scan the Packages or Library folders?**

No. MetaGuard scans the `Assets/` folder only. `Packages/`, `Library/`, and `Temp/` are not included.

---

**What does the health score mean?**

The health score (0–100) is a weighted aggregate of all detected issues. A score of 100 means no issues were found. Lower scores reflect the number and severity of detected problems. The score is a quick indicator — always review the Issues tab for specifics.

---

## Simulation

**What is the difference between Dangerous and Destructive verdicts?**

A **Dangerous** verdict means the operation has potential side effects that warrant review before applying. MetaGuard will apply Dangerous operations if you confirm the Apply dialog.

A **Destructive** verdict means the operation will break existing references. MetaGuard does not apply Destructive operations through Apply or Fix All Safe — they require explicit handling.

---

**Can I run Simulate multiple times without applying?**

Yes. Simulate can be run as many times as needed without affecting the project. Run Scan + Analyze first if the project state has changed since the last scan.

---

## Applying

**What happens if Unity closes or crashes during Apply?**

The three-tier safety system handles it. Each file write is staged through a temporary path — if the write is interrupted, the original file is untouched. On the next Editor open, if the session is still within 48 hours, Rollback is available to restore all files to their pre-Apply state.

---

**What is the difference between Apply and Fix All Safe?**

**Apply** runs the operations from the most recent simulation, writing Safe and Warning verdicts to disk.

**Fix All Safe** runs the full pipeline — scan, simulate, and apply — in one automated action, writing only Safe verdicts. Use it for routine maintenance when you want a conservative, fully automated pass.

---

**Can I apply individual operations rather than all at once?**

Not in v1.0.0. Apply writes all Safe and Warning operations from the most recent simulation. Use the Issues tab filters before simulating to narrow the scope if you want to limit which issues are addressed.

---

## Rollback

**How long is rollback available?**

48 hours from the time Apply completed, including across editor restarts and domain reloads.

---

**Does rollback undo manual changes I made after Apply?**

Yes. Rollback restores all affected files to their pre-Apply state. Any changes made to those files after Apply — manual edits, Unity reimports, other tooling — are overwritten. Only the files that were part of the Apply session are affected.

---

**Can I roll back after restarting Unity?**

Yes. The session persists across editor restarts for 48 hours.

---

## Cache

**Should I commit `scan_cache.txt` to source control?**

No. The cache file is machine-specific and should be excluded from version control. Each team member builds their own cache from their first local scan.

---

**Is cache data shared across the team?**

No. Cache data is local to the machine. Different team members will have different cache states depending on their local scan history.

---

**Does disabling the cache affect scan accuracy?**

No. With the cache disabled, every file is read fresh regardless of modification state. Accuracy is identical — only performance differs. The cache never alters what is detected, only what is skipped.
