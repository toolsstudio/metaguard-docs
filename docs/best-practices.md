# Best Practices

Recommendations for teams using MetaGuard in production projects.

---

## Source Control Configuration

**Commit:**
- `Assets/MetaGuard/metaguard_policy.json` — shared policy for all team members and CI
- `Assets/MetaGuard/health_log.json` — optional; enables historical health tracking across the team

**Do not commit:**
- `Assets/MetaGuard/Snapshots/` — session-specific, can be large
- `Assets/MetaGuard/MetaGuardReports/` — CI artifacts, managed externally
- `Assets/MetaGuard/scan_cache.txt` — machine-specific

`.gitignore` entries to add:

```
Assets/MetaGuard/Snapshots/
Assets/MetaGuard/MetaGuardReports/
Assets/MetaGuard/scan_cache.txt
```

MetaGuard includes these entries in its own `.gitignore` by default. Verify they have not been removed after an update.

---

## Running Scans

**Run a scan before committing.** GUID issues created by one developer compound when others pull and run. A single missing `.meta` can trigger a cascade of broken references across the project. Catching issues before commit keeps the main branch clean.

**Scan after merges.** Git merge conflicts in `.meta` files are a common source of GUID corruption. Run a scan immediately after resolving a conflict that touched `.meta` files. Pay particular attention to GUIDCollision results — these almost always indicate a merge that incorrectly combined two conflicting `.meta` files into one, preserving both original GUIDs.

**Do not scan while Unity is importing.** If Unity is processing an import (the progress bar is visible in the bottom-right corner), wait for it to complete before running a MetaGuard scan. Assets mid-import may have incomplete GUID data, which will produce false or incomplete results.

**Use the cache for day-to-day scans.** The cache significantly reduces scan time on projects with large, stable asset sets. Disable it only when you need a guaranteed full re-read — for example, after a file operation performed outside of Unity that may not have updated modification timestamps.

---

## Applying Fixes

**Always Simulate before Apply.** The simulation verdict for every operation is visible in the Results tab. Review Dangerous and Destructive verdicts carefully — they indicate operations that MetaGuard cannot verify are safe without project-specific knowledge.

**Use Fix All Safe for routine maintenance.** Fix All Safe applies only Safe verdicts, runs the full pipeline automatically, and creates a snapshot before writing. It is the right choice for addressing accumulated OrphanedMeta, ZeroGUID, and MalformedGUID issues in a project that is already in a known-good state.

**Apply manually when the issue set is unfamiliar.** If a scan reveals a large number of BrokenReference or CyclicDependency issues on a project you haven't worked on recently, take the time to review each issue before applying. Simulation verdicts tell you the technical safety of an operation — only you know whether a referenced asset was intentionally deleted.

**One Apply session per workflow.** MetaGuard's rollback is session-based. If you run Apply, then run Apply again on a different set of issues, the Rollback button restores only the most recent session. If you need to be able to roll back both sessions independently, use a source control snapshot (branch, stash, or commit) in addition to MetaGuard's built-in rollback.

**Commit before a significant Apply.** The snapshot system provides a safety net for the immediate Apply session, but it does not replace source control. On large projects or after a merge that touched many `.meta` files, commit first so you can recover fully if something unexpected happens.

---

## Policy Configuration

**Block GUIDCollision from the start.** GUID collisions cause non-deterministic asset loading that is difficult to reproduce and harder to debug. The default policy blocks them in CI — keep this setting.

**Promote Flagged issues to AutoFix incrementally.** Start with the defaults. Once the team is confident that OrphanedMeta and MalformedGUID fixes are always safe in your project, they will already be on AutoFix. Consider promoting MissingMeta to AutoFix only after confirming that your team always deletes assets through the Unity Editor (which removes the `.meta` automatically) rather than via the filesystem.

**Use excludeAssetPaths for known false positives.** If a specific plugin or package produces persistent BrokenReference false positives that cannot be resolved, add the asset path to `excludeAssetPaths` rather than setting `BrokenReference` to `Ignore`. Path-level exclusion is specific and reversible; global Ignore is neither.

**Keep the policy file reviewed and minimal.** Every `Ignore` entry reduces MetaGuard's effective detection surface. Review ignored issue classes periodically to confirm they are still intentional.

---

## CI Integration

**Set a health score gate, not just a violation flag.** The `--threshold` argument allows you to fail a build when the health score drops below a specific value, even if no individual issue is classified as Critical or High. A health score gate catches accumulation of Low-severity issues before they become significant.

**Store reports as CI artifacts.** MetaGuard writes JSON reports to `MetaGuardReports/` automatically. Upload this directory as a build artifact on every CI run — it provides a searchable history of project health over time and makes it easy to compare before and after a large refactor.

**Run MetaGuard in a separate CI job.** MetaGuard scans complete quickly on typical projects. Running it as a dedicated job with its own artifact upload keeps it independent of the build outcome and ensures a report is always generated even when the build fails.

**Block merges on GUIDCollision.** Configure your CI to enforce exit code 0 (or the absence of `has_violations: true` in the report) as a merge requirement. Even if you use a lenient threshold for other issue classes, GUIDCollision should always be a hard gate.

---

## Large Projects

**Keep the cache enabled.** On projects with thousands of assets, the difference between a cached and uncached scan can be significant. Only disable the cache when you have a specific reason.

**Exclude generated asset directories.** If your project generates assets programmatically (addressable bundles, LOD outputs, procedurally generated content) into a known directory, consider whether those paths need to be scanned. Assets in generated directories often have short lifecycles and produce OrphanedMeta results that are not meaningful. Add them to `excludeAssetPaths` if the volume of false positives is high.

**Schedule regular scans in long-running development phases.** On projects with multiple developers working concurrently, GUID issues accumulate gradually. A weekly scan in CI — even outside the main PR gate — provides early warning of accumulating technical debt and prevents small issues from compounding into larger ones before a release.

**Review health score trends, not just individual scans.** Commit `health_log.json` to source control and review it periodically. A health score that is gradually declining indicates that issues are being introduced faster than they are being fixed — a signal to schedule a dedicated cleanup pass before the score reaches a critical level.
