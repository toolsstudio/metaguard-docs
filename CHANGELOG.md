# Changelog

All notable changes to MetaGuard are documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).  
MetaGuard adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

---

## [1.0.0] — 2026-04-24

### Added

#### Scan & Analysis
- Full project enumeration with progress reporting and cancellation support
- Dependency graph construction from asset YAML data
- Seven-class issue detection: GUID collisions, zero GUIDs, malformed GUIDs, orphaned `.meta` files, missing `.meta` files, broken references, and cyclic dependencies
- Risk classification per issue: Critical, High, Medium, Low — based on downstream reference count and fix complexity
- Project health score (0–100) reported after each scan
- Self-exclusion filter — MetaGuard's own assets are never included in scan results

#### Safe Apply System
- Simulation engine: every fix operation tested against an in-memory graph clone before any file is written
- Simulation verdicts: Safe, Warning, Dangerous, Destructive
- Apply writes only simulation-approved operations
- Atomic write staging — each write committed through a temp file; original is never corrupted on interrupt
- Unity asset database refresh batched after all writes complete

#### Rollback System
- Pre-apply snapshot written to `MetaGuard/Snapshots/` before the first file write
- Session persists across domain reloads and editor restarts via persistent store
- SHA-256 integrity validation before any restore
- Automatic rollback triggered on any apply exception — project never left in partial state
- Snapshot archive retained for 48 hours

#### Scan Cache
- Tracks file modification times to skip unchanged assets on repeated scans
- Toggle in the MetaGuard toolbar: `Cache ON` / `Cache OFF` (default: ON)
- Cache data stored in `MetaGuard/scan_cache.txt`, persists across editor restarts

#### Auto-Fix Pipeline
- Prioritized fix suggestion generation per detected issue
- Non-conflicting operations grouped into ordered fix batches
- **Fix All Safe** — applies all simulation-verified Safe operations in one action

#### UI
- Five-stage pipeline step bar: Scan → Analyze → Simulate → Apply → Rollback
- **Issues tab** — search, risk filter, type filter, foldable groups, issue inspector
- **Suggestions tab** — per-suggestion confidence, priority, auto-eligibility, operation detail
- **Results tab** — simulation verdicts, apply write records, auto-fix summary
- **Log tab** — ring-buffered structured log (2,000 entries), copy to clipboard, clear
- Confirmation dialog required before Apply and Fix All Safe

---

[Unreleased]: https://github.com/ToolsStudio/MetaGuard-Docs/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/ToolsStudio/MetaGuard-Docs/releases/tag/v1.0.0
