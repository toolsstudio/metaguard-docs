> **Archive — MetaGuard 1.x.** This version has reached end of life. Current documentation is at the [repository root](../../../README.md).

---

# Troubleshooting

---

## The MetaGuard Menu Item Does Not Appear

**Check for compilation errors.**

The `Tools > MetaGuard` menu is registered by `[MenuItem]` attributes in the MetaGuard Editor assembly. If the assembly fails to compile for any reason, no menu items are registered. Open the Unity Console and resolve any compilation errors before checking the menu again.

Common causes:
- A file was not imported correctly — re-import the `.unitypackage` and confirm all items are checked in the import dialog
- The project is on a Unity version below 2020.3 LTS — MetaGuard requires 2020.3 or later
- A stray copy of `ConflictDetector.cs` exists at `Assets/ConflictDetector.cs` (outside the MetaGuard Editor assembly) — see [Stray File Causing Compile Errors](#stray-file-causing-compile-errors-cs0246-cs0234) below

**Verify the import was complete.**

Go to **Assets > Import Package > Custom Package…**, re-select the `.unitypackage`, and in the import dialog confirm that all files under `Assets/MetaGuard/Editor/` are checked before clicking Import.

---

## Stray File Causing Compile Errors (CS0246 / CS0234)

**Symptoms:**

```
Assets\ConflictDetector.cs: error CS0246: The type or namespace name 'FixSuggestion' could not be found
Assets\ConflictDetector.cs: error CS0234: The type or namespace name 'Internal' does not exist in the namespace 'ToolsStudio.MetaGuard'
Assets\MetaGuard\Editor\MetaGuardWindow.cs: error CS0246: The type or namespace name 'ConflictDetector' could not be found
```

**Cause:**

A copy of `ConflictDetector.cs` was placed at `Assets\ConflictDetector.cs` — outside the MetaGuard Editor assembly. Unity compiles root-level files into `Assembly-CSharp`, which does not reference `MetaGuard.Internal`. Every `FixSuggestion` usage in that file fails to resolve, and `MetaGuardWindow` in the Editor assembly cannot see the type.

**Fix:**

Delete the stray file:
```
Assets\ConflictDetector.cs   ← DELETE THIS
```

The correct copy is at:
```
Assets\MetaGuard\Editor\Automation\ConflictDetector.cs   ← keep this
```

**Prevention:**

MetaGuard includes `MetaGuardStrayFileGuard.cs` — an `[InitializeOnLoad]` class that runs at every domain reload and logs a `LogError` with the exact path and fix instruction if the stray file is ever reintroduced.

---

## Scan Returns Zero Issues on a Project with Known Problems

**Check the scan cache.**

The cache skips unchanged files. If a file was corrupted or modified outside of Unity without changing its modification timestamp, the cache will not detect the change.

Disable the cache (Cache OFF toggle in the toolbar) and run the scan again. If the correct issues appear, the cache was stale. Re-enable the cache after the scan.

**Check the policy.**

Open the **Policy** tab and confirm the relevant issue classes are not set to `Ignore`. If `excludeAssetPaths` contains paths you expect to be scanned, remove them and click **Reload**.

**Wait for pending imports to complete.**

If Unity is mid-import when a scan starts, assets being processed may have incomplete GUID data. Wait for the import progress bar (bottom-right of the Editor) to clear before scanning.

---

## Scan Returns Many BrokenReference Issues on a URP Project

This is addressed by the policy system's `excludeAssetPaths` feature, introduced in 2.x. See the [2.x troubleshooting guide](../../../docs/troubleshooting.md#scan-returns-120-brokenreference-issues-on-a-urp-project).

## Apply Writes Files But Unity Does Not Reflect the Changes

Apply triggers `AssetDatabase.Refresh` after all writes complete. If the changes are not visible in the Editor:

1. Run **Scan + Analyze** again — the new scan will pick up the current disk state
2. If the issue persists, go to **Assets > Refresh** in the Unity menu to force a manual full refresh
3. If an asset still appears as the old version in the Inspector, select it and press **Ctrl+R** / **Cmd+R** to force a reimport

---

## Rollback Does Not Restore All Files

Each snapshot entry is validated by SHA-256 hash before restoration. If a snapshot file is missing or its hash does not match the stored value, MetaGuard skips that file and logs an error identifying the path. The remaining files in the session are still restored.

Common causes:
- `Assets/MetaGuard/Snapshots/` was committed to version control and line endings were normalized — add the directory to `.gitignore`
- The snapshot file was deleted or modified after Apply
- The session is older than 48 hours (expired and pruned automatically)

Files that could not be restored must be recovered from source control.

---

## Rollback Button is Disabled After Apply

The Rollback button is only enabled when a valid snapshot session exists within the 48-hour window. If the button is disabled:

- The session may have expired (more than 48 hours since Apply)
- The session record in `Snapshots/` may have been deleted
- The apply may not have written any files (if all operations were already correct)

If the session was valid but the button is disabled after a domain reload, wait for Unity to finish domain initialization and check again. The session initializer runs on `delayCall` during startup.

---

## YAML Parsing Warnings

YAML parsing warnings for demo test assets are a 2.x feature. See the [2.x troubleshooting guide](../../../docs/troubleshooting.md).

## Health Score Drops Significantly After a Routine Apply

Apply writes only Safe simulation verdicts. A score drop after Apply means the applied fixes revealed previously hidden issues. For example, regenerating a GUID for a zero-GUID asset updates the asset's GUID in its own `.meta` file, but other assets that happened to reference the old (zero) GUID now reference a GUID that does not exist — appearing as BrokenReference issues.

This is expected behaviour — the original zero-GUID state was masking the broken references. Run another **Scan + Analyze** to see the new issue set, then Simulate and Apply the next round of fixes.

---

## Support

If the issue is not covered here:

- **Discord**: [discord.gg/rYbZZz5GH4](https://discord.gg/rYbZZz5GH4) — primary support channel
- **Bug reports**: [discord.gg/mQYguyhYwA](https://discord.gg/mQYguyhYwA)
- **Email**: tools.studio@zohomail.in

When reporting, include:
- MetaGuard version (shown in the About tab)
- Unity version and OS
- Relevant Console output
- Steps to reproduce
