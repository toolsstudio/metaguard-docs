> ⚠️ **This is an archived version (v1). Use [MetaGuard Pro (v2)](../../versions/v2/) documentation instead.**

---

# Changelog — MetaGuard 1.x

> This is the archived changelog for MetaGuard 1.x.
> For the current changelog (2.x), see the [main CHANGELOG](../../CHANGELOG.md).

---

## [1.0.0] — 2026-04-24

### Added

- Full project enumeration with background scan, progress reporting, and cancellation support
- Dependency graph construction from asset YAML serialization data
- Seven-class issue detection: GUID collisions, zero GUIDs, malformed GUIDs, orphaned `.meta` files, missing `.meta` files, broken references, cyclic dependencies
- Risk classification per issue: Critical, High, Medium, Low
- Project health score (0–100) reported after each scan
- Self-exclusion filter — MetaGuard's own assets excluded from scan results
- Simulation engine with four verdicts: Safe, Warning, Dangerous, Destructive
- Atomic write staging — each file write committed through a temporary path
- Pre-apply snapshot with SHA-256 integrity validation
- Persistent session archive — rollback available for 48 hours, survives editor restarts
- Automatic failure rollback — project never left in partial state
- Scan cache with modification-time tracking, togglable in toolbar
- Fix All Safe — full pipeline (scan, simulate, apply) in one action, Safe verdicts only
- Five-stage pipeline step bar with stage-gate enforcement
- Issues tab with search, filter, foldable groups, per-issue inspector
- Suggestions tab with confidence, priority, auto-eligibility, and operation detail
- Results tab with simulation verdicts and apply write records
- Log tab — ring-buffered structured log, 2,000 entries, copy to clipboard

[1.0.0]: https://github.com/toolsstudio/metaguard/releases/tag/v1.0.0
