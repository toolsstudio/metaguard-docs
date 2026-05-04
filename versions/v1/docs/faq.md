> ⚠️ **This is an archived version (v1). Use [MetaGuard Pro (v2)](../../versions/v2/) documentation instead.**

---

# FAQ

---

## General

**Does MetaGuard modify anything at runtime?**

No. All MetaGuard assemblies are Editor-only and are excluded from builds automatically. Nothing from MetaGuard appears in a player build. The `MetaGuard.Internal` assembly is runtime-safe (no Editor dependencies) but is marked `autoReferenced: true` so the scene serializer can locate the `DemoBootstrap` MonoBehaviour — it does not introduce any runtime code to builds.

**Does MetaGuard work with URP, HDRP, and the built-in render pipeline?**

Yes. MetaGuard operates on `.meta` files and the GUID layer of the asset system, which is render-pipeline-independent. URP projects may initially show false-positive BrokenReference results for pipeline-managed assets — this is addressed by the `excludeAssetPaths` field in the policy file. The default policy already excludes the relevant URP assets.

**What Unity versions are supported?**

Unity 2020.3 LTS and later, including Unity 6. Earlier versions are not tested and are not supported.

**Is MetaGuard safe to use on a project in active development?**

Yes. MetaGuard creates a pre-apply snapshot before any file write and provides a 48-hour rollback window. The simulation stage confirms every operation's safety before Apply runs. No file is modified without a recoverable backup.

**Does MetaGuard require any dependencies?**

No. MetaGuard has no external package dependencies and does not require the Package Manager to install anything. All required code is included in the `.unitypackage`.

---

## Scanning

**How long does a scan take?**

On a project with a few hundred assets, a full scan typically completes in under one second. With the cache enabled, repeated scans of unchanged projects complete in milliseconds. Large projects (10,000+ assets) may take 5–15 seconds on a cold cache, depending on disk speed.

**Does MetaGuard scan files in the `Packages/` directory?**

The default scan depth is `AssetsAndPackages`, which indexes GUIDs in the `Packages/` folder. This is required to correctly resolve references from assets like URP materials that point to shader GUIDs inside the `Packages/` directory. MetaGuard never writes to `Packages/`.

**Why does my project show 120+ BrokenReference issues on the first scan after upgrading from v1.0.0?**

In v1.0.0, the default scan depth was `AssetsOnly`. Upgrading to v2.0.0 changes the default to `AssetsAndPackages`, which may reveal pre-existing issues that were not visible before because the relevant GUIDs were outside the scan boundary.

Additionally, if your policy file was created in v1.0.0, it will not contain the `excludeAssetPaths` entries for URP assets. Add them manually:

```json
"excludeAssetPaths": [
  "Assets/DefaultVolumeProfile.asset",
  "Assets/UniversalRenderPipelineGlobalSettings.asset"
]
```

Then reload the policy and run the scan again.

**Does the scan include scripts?**

MetaGuard scans `.meta` files and reads GUID references from YAML-serialised asset files. It does not parse C# script contents. Scripts are included in the scan to the extent that their `.meta` files are checked for GUID integrity (zero GUID, malformed GUID, etc.).

**Can I exclude specific folders from scanning?**

At the file level, yes — add paths to `excludeAssetPaths` in the policy file. Full folder exclusion is not currently available as a configuration option. Paths in `excludeAssetPaths` apply to BrokenReference analysis specifically. Other issue classes (OrphanedMeta, ZeroGUID, etc.) are not filtered by this field.

**What does the Dirty badge mean?**

The `Dirty (N)` badge in the header indicates that N assets have changed since the last scan. MetaGuard monitors the project for file changes passively. The Dirty state is a prompt to run a new scan — it does not affect existing scan results until you click Scan + Analyze.

---

## Apply & Rollback

**Is it safe to close the Editor immediately after Apply?**

Yes. Session data is written to disk before any project file is modified. You can close the Editor at any point after Apply, and Rollback will still be available when you reopen the project (within the 48-hour window).

**Can I roll back a Fix All Safe operation?**

Yes. Fix All Safe creates the same pre-apply snapshot as a manual Apply. Rollback works identically for both.

**What happens if rollback fails for one file?**

MetaGuard validates each file by SHA-256 hash before restoring it. If a hash mismatch or missing snapshot file is detected, MetaGuard logs an error for that specific file and continues restoring the rest. A partial rollback is better than an aborted one. Files that could not be restored must be recovered from source control.

**Does rollback work across Unity restarts?**

Yes. Sessions persist on disk and survive domain reloads, editor restarts, and Unity crashes. The session initializer checks for valid sessions on startup and activates the Rollback button if one is found within the 48-hour window.

**Can I undo a Rollback?**

No. Rollback is a one-way operation. Once completed, the session is cleared and the pre-rollback state is gone from MetaGuard's snapshot system. If you need to revert a rollback, use source control.

**Why does Apply require a Simulate first?**

The simulation stage ensures that every operation has a known verdict before any file is touched. Without simulation, MetaGuard cannot guarantee that an operation is Safe. This gate is enforced at the engine level and cannot be bypassed.

---

## Policy

**What happens if the policy file is missing?**

MetaGuard applies a set of built-in defaults and continues normally. The defaults match those in the auto-generated policy file. MetaGuard does not fail or show an error if the policy file is absent — it simply falls back to defaults.

**Can different team members use different policy settings?**

The policy file is committed to source control and shared across the team. This is the intended model. If you need developer-specific overrides, create a local policy file and point the CLI to it using `--policy <path>`. The Editor tool always reads from `Assets/MetaGuard/metaguard_policy.json`.

**What does the `Block` action do in the Editor?**

In the Editor, `Block` behaves identically to `Flag` — the issue is surfaced and a suggestion is generated. The distinction is in the CLI: `Block` forces `has_violations = true` and exit code 1 regardless of issue counts or threshold. This is what makes GUIDCollision a hard CI gate while still allowing developers to review the issue in the Editor.

**Can I add custom issue classes to the policy?**

No. The policy file governs the seven built-in issue classes only. Custom detection logic is not supported in the current version.

---

## CLI

**Does the CLI apply fixes?**

No. The CLI scans, analyzes, and reports. It does not apply any fix operations. Fix operations must be performed through the Editor. This is an intentional constraint — automated writes to project files in a CI environment, without a human review step and a rollback path, would be unsafe.

**Can I run the CLI without a Unity license?**

No. The CLI runs inside Unity batch mode, which requires an active Unity license. MetaGuard does not impose additional licensing beyond what Unity requires. In CI environments, a Unity floating license or a seat assigned to the CI machine is required.

**The CLI exits with code 2. What does that mean?**

Exit code 2 indicates a scan-level failure — not a GUID integrity violation, but an error that prevented the scan from completing. Common causes are an invalid project path, a missing Unity license, or malformed JSON in the policy file. Check the `-logFile` output and the report's `error` field for the specific failure message.

**Can I use the CLI with a custom policy file that is not in the project?**

Yes. Use `--policy <absolute-or-relative-path>` to point the CLI at any valid `metaguard_policy.json` file, regardless of its location. This is useful for CI environments that manage policy files externally or apply different rules for different pipeline stages.

**Why does the CLI always generate a report even when `--output` is not specified?**

Consistent report generation makes it possible to upload CI artifacts unconditionally — you do not need to handle the case where no report was produced. The auto-report path (`MetaGuardReports/report_YYYY-MM-DD_HH-mm-ss.json`) is deterministic and does not overwrite previous reports.

---

## Demo System

**What is the demo system for?**

The demo system seeds a controlled set of broken asset states so you can verify that MetaGuard detects, simulates, and fixes each issue class correctly in your environment. It is useful for verifying a fresh installation, onboarding new team members, and regression testing after a package update.

**Why does Validate Demo State show 3/5 after setup?**

Unity's `AssetDatabase.Refresh()` regenerates missing `.meta` files (TC-2) and processes orphaned `.meta` files (TC-1) automatically. In MetaGuard 2.0.0, TC-1 and TC-2 are written as the last step of setup, after all `Refresh()` calls complete, so Unity has no opportunity to repair them. If you are seeing 3/5, you are likely running an earlier version. Update to the latest package.

**Are the YAML warnings for TC-3 and TC-4 expected?**

Yes. TC-3 contains an all-zero GUID and TC-4 contains a non-hex GUID string. These are deliberately invalid values. The Unity YAML parser warnings confirm the test cases are in the correct broken state. They are not MetaGuard errors and require no action.

**Does the demo affect my project permanently?**

No. All test case files are written to `Assets/MetaGuard/Demo/TestAssets/`. Running **Teardown Demo Assets** deletes all seeded files and returns the directory to a clean state. The demo scene, scripts, and assembly definitions are preserved — they are package files, not test state.

---

## Licensing and Purchase

**Where can I purchase MetaGuard?**

MetaGuard 2.0.0 is available on the [Unity Asset Store](https://assetstore.unity.com/packages/slug/376206).

**Is there a free trial or demo version?**

There is no separate trial version. The demo system included in the package allows you to validate the tool in your project environment after purchase.

**What is the license?**

MetaGuard is licensed under the Unity Asset Store End User License Agreement (EULA). See the [LICENSE](../LICENSE) file and the Asset Store product page for details.

**Are updates included?**

Updates are distributed through the Unity Asset Store. Minor updates (bug fixes, compatibility patches) within the current major version are included at no additional cost. Check the Asset Store product page for update availability.
