# Changelog

All notable changes to MetaGuard are documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
MetaGuard adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

---

## [2.0.0] — 2026-05-01

### Added

#### Policy System
- `metaguard_policy.json` — per-project policy file controlling how each issue class is handled
- Four policy actions: `AutoFix`, `Flag`, `Ignore`, `Block`
- `Block` action enforces a hard violation in CLI scans (exit code 1) regardless of other counts
- `excludeAssetPaths` array — project-relative paths excluded from BrokenReference analysis, used to suppress URP global settings and other pipeline-managed assets that produce false positives
- Policy tab in the MetaGuard window — displays current rules, action legend, and reload button
- `Tools > MetaGuard > Create Default Policy File` menu item
- Policy enforcement consistent between Editor tool and CLI — same file governs both

#### CLI / CI Integration
- `MetaGuardCLI.Run` — headless entry point for Unity batch mode
- JSON and text report formats
- Auto-report: when `--output` is not specified, reports are written automatically to `MetaGuardReports/report_YYYY-MM-DD_HH-mm-ss.json` at the project root
- CLI arguments: `--policy <path>`, `--format json|text`, `--output <path>`, `--threshold <0–5>`, `--version`, `--help`
- Deterministic exit codes: `0` = clean, `1` = violations, `2` = scan failure
- Violation logic: `has_violations = true` when `critical > 0`, `high > 0`, `health_score == 0`, or `health_score <= threshold`
- Debug line emitted after each evaluation: `ViolationCheck => critical:X high:Y threshold:Z result:true/false`
<<<<<<< HEAD
- Shell script template included at `Assets/MetaGuard/CI/metaguard_scan.sh`
=======
- Shell script template included at `Assets/CI/metaguard_scan.sh`
>>>>>>> 535bab848ab29badabd4510c97f00570ec3ca11a
- GitHub Actions usage example in documentation

#### History Tab
- Records health score and issue counts from every completed scan
- Scores persisted to `MetaGuard/health_log.json` — commit to track health over time
- Bare version numbers displayed (no `v` prefix)

#### Demo System
- Five reproducible test cases: OrphanedMeta (TC-1), MissingMeta (TC-2), ZeroGUID (TC-3), MalformedGUID (TC-4), BrokenReference (TC-5)
- `DemoSetupEditor` — four menu items: Setup Demo Assets, Run Demo Scan, Validate Demo State, Teardown Demo Assets
- Setup is deterministic: TC-1 and TC-2 written after all `AssetDatabase.Refresh()` calls complete, preventing Unity from auto-repairing test states
- Scene written as raw YAML — avoids `EditorSceneManager` "same open scene" assertion
- Assembly split: `MetaGuard.Demo` (runtime) and `MetaGuard.Demo.Editor` (Editor-only)
- `MetaGuardStrayFileGuard` — `[InitializeOnLoad]` guard detects misplaced `ConflictDetector.cs` at domain reload

#### Thread Safety
- `ThreadSafeProgress<T>` replaces `System.Progress<T>` in the scan pipeline — zero Unity API calls from worker threads, eliminates `JobTempAlloc` leak warnings
- `EditorApplication.update` pump reads volatile progress fields on the main thread
- All background tasks stored as fields; `OnBeforeAssemblyReload` cancels in-flight work with a 2-second grace period before domain teardown
- `HealthHistoryStore.FlushPendingWarnings()` — background errors written to a volatile field, drained and logged on the main thread each frame

#### About Tab
- Redesigned layout: title, version, publisher metadata, and five action buttons only
- Buttons: Documentation, Rate on Asset Store, Discord Support, Email Support, Report a Bug (Discord)
- No inline documentation — all links point to external resources

### Changed

- **Scan depth default**: Changed from `AssetsOnly` to `AssetsAndPackages` — indexes URP and other package GUIDs, eliminating false-positive BrokenReference reports from package-managed assets
- **Header version format**: Displays `Version 2.0.0` (previously `v2.0.0`)
- **History tab version format**: Bare numbers without prefix (previously `v2.0.0`)
- **`PolicyFileLoader.CreateDefaultFromMenu`**: Shows a dialog if the file already exists rather than logging to Console
- **`ScanConfiguration.Default()`**: Uses `ScanDepth.AssetsAndPackages`
- **`IntegrityAnalyzer.Analyze()`**: Accepts an optional `PolicyConfig` parameter; `BrokenReference` results filtered against `excludeAssetPaths` before surfacing
- MetaGuard is now a **paid asset**; available exclusively on the Unity Asset Store

### Fixed

- CLI violation logic: `high > 0` now correctly sets `has_violations = true` (previously only threshold-based evaluation was applied, meaning high-risk issues did not trigger a violation unless they exceeded the threshold)
- `health_score == 0` always produces a violation regardless of issue counts or threshold
- CI shell script uses `--output` flag (file-write mode) rather than stdout redirect, preventing log and report stream interleaving in Unity batch mode
- `GetWindow<MetaGuardWindow>(false)` — named parameter `focus:` corrected to positional bool in `DemoSetupEditor`
- `_pendingWarning` field declaration was missing from `HealthHistoryStore`, causing CS0103 compile error

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

[Unreleased]: https://github.com/toolsstudio/metaguard/compare/v2.0.0...HEAD
[2.0.0]: https://github.com/toolsstudio/metaguard/compare/v1.0.0...v2.0.0
[1.0.0]: https://github.com/toolsstudio/metaguard/releases/tag/v1.0.0
