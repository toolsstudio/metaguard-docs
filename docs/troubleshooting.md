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

## Scan Returns 120+ BrokenReference Issues on a URP Project

URP's `UniversalRenderPipelineGlobalSettings.asset` and `DefaultVolumeProfile.asset` reference GUIDs in the `Packages/` folder. These references are valid at runtime but may appear broken if the scanner does not index the Packages directory or if those specific assets are not excluded from analysis.

MetaGuard 2.0.0 defaults to `AssetsAndPackages` scan depth and excludes these assets in the default policy. If you are still seeing these false positives after updating to 2.0.0:

1. Open `Assets/MetaGuard/metaguard_policy.json`
2. Confirm `excludeAssetPaths` contains both paths:
   ```json
   "excludeAssetPaths": [
     "Assets/DefaultVolumeProfile.asset",
     "Assets/UniversalRenderPipelineGlobalSettings.asset"
   ]
   ```
3. Open the **Policy** tab and click **Reload**
4. Run **Scan + Analyze** again

If the policy file does not contain those entries (for example, if you are using a policy file created in v1.0.0), add them manually and reload.

---

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

## YAML Parsing Warnings for TC-3 and TC-4

```
The GUID inside 'Assets/MetaGuard/Demo/TestAssets/TC3_ZeroGuidAsset.txt.meta' cannot be extracted by the YAML Parser.
The .meta file TC3_ZeroGuidAsset.txt.meta does not have a valid GUID...
```

These warnings appear after running **Setup Demo Assets** from the Demo system. They are **expected and intentional** — TC-3 and TC-4 are test cases with deliberately invalid GUIDs. The YAML parser warnings confirm the test cases are in the correct broken state. They are not MetaGuard errors.

---

## `m_DisallowAutoRefresh >= 0` Assertion Errors

These assertion errors fire when `AssetDatabase.AllowAutoRefresh()` is called more times than `AssetDatabase.DisallowAutoRefresh()`. MetaGuard 2.0.0 does not call either of these methods. If you see these errors after running MetaGuard:

1. Confirm you are on the latest version — earlier builds of the Demo system contained an imbalanced call pair that was removed in 2.0.0
2. If the error persists, check whether another tool in the project is calling `DisallowAutoRefresh` without a corresponding `AllowAutoRefresh`

---

## "Overwriting the Same Path as Another Open Scene" Error

This error fires when `EditorSceneManager.SaveScene` targets a scene that is already loaded. MetaGuard 2.0.0 writes the demo scene as raw YAML and does not call `EditorSceneManager.SaveScene`. If you see this error:

1. Confirm you are on the latest version — earlier builds used `EditorSceneManager` for scene creation
2. Reimport the package to ensure `DemoSetupEditor.cs` is the current version

---

## Demo Test Cases TC-1 and TC-2 Show NOT READY After Setup

`AssetDatabase.Refresh()` regenerates missing `.meta` files (TC-2) and processes orphaned `.meta` files (TC-1) automatically. In MetaGuard 2.0.0, TC-1 and TC-2 are written after all `Refresh()` calls complete, leaving Unity with no opportunity to repair them during setup.

If you see NOT READY states:

1. Run **Teardown Demo Assets** to clear the test directory
2. Run **Setup Demo Assets** again
3. Run **Validate Demo State** immediately after — before any other Unity operation triggers a refresh

If NOT READY persists after these steps, confirm you are running the latest package version.

---

## CLI Exits with Code 2

Exit code 2 indicates a scan-level failure rather than a GUID integrity violation.

Common causes:
- The project path passed to `-projectPath` does not exist or is not a valid Unity project
- Unity failed to open the project — check the `-logFile` for Unity startup errors
- The MetaGuard policy file exists but contains malformed JSON — validate `metaguard_policy.json` with a JSON linter
- Unity license is not activated for the machine running the CLI scan

The report file will contain an `error` field identifying the specific failure when exit code 2 is produced.

---

## Health Score Drops Significantly After a Routine Apply

Apply writes only Safe simulation verdicts. A score drop after Apply means the applied fixes revealed previously hidden issues. For example, regenerating a GUID for a zero-GUID asset updates the asset's GUID in its own `.meta` file, but other assets that happened to reference the old (zero) GUID now reference a GUID that does not exist — appearing as BrokenReference issues.

This is expected behaviour — the original zero-GUID state was masking the broken references. Run another **Scan + Analyze** to see the new issue set, then Simulate and Apply the next round of fixes.

---

## Policy Log Appears on Every Editor Startup

```
[MetaGuard] Policy loaded from Assets/MetaGuard/metaguard_policy.json
```

This log entry appears when the MetaGuard window opens. It is a structured informational log and is intentional — it confirms which policy file is active. It should appear once per MetaGuard window open event.

If it appears repeatedly on a single startup (multiple times in quick succession), check whether the window is being opened programmatically more than once — for example, by a Demo menu item followed by a manual open.

---

## Support

If the issue is not covered here:

- [Join our Discord](https://discord.gg/rYbZZz5GH4)
- [Report a bug](https://discord.gg/mQyughywA)
- [Email support](mailto:tools.studio@zohomail.in)

---

### When reporting, include:

- MetaGuard version (shown in the About tab)
- Unity version and OS
- Relevant Console output
- Steps to reproduce
