# Demo System

The demo system seeds a controlled set of broken asset states so you can verify that MetaGuard detects, simulates, and fixes them correctly in your environment.

---

## Purpose

The demo is useful for:

- Verifying a fresh installation works end-to-end
- Onboarding developers who are new to MetaGuard
- Regression testing after a MetaGuard package update
- Confirming the scan pipeline correctly handles each issue class

The test cases are intentionally broken assets. They are not real project issues and will not affect your project outside the `Assets/MetaGuard/Demo/TestAssets/` directory.

---

## Menu Items

```
Tools > MetaGuard > Setup Demo Assets
Tools > MetaGuard > Run Demo Scan
Tools > MetaGuard > Validate Demo State
Tools > MetaGuard > Teardown Demo Assets
```

---

## Setup

```
Tools > MetaGuard > Setup Demo Assets
```

Creates five test case files in `Assets/MetaGuard/Demo/TestAssets/` and writes the demo scene. The setup is deterministic and idempotent — running it multiple times produces the same state.

After setup, run **Validate Demo State** to confirm all five cases are in the correct broken state before scanning.

---

## Test Cases

| ID | Type | File | What MetaGuard Detects |
|---|---|---|---|
| TC-1 | OrphanedMeta | `TC1_OrphanedMeta.meta` (no asset file) | `.meta` file with no corresponding asset. MetaGuard proposes deletion. Policy action: AutoFix. |
| TC-2 | MissingMeta | `TC2_MissingMeta.txt` (no `.meta` file) | Asset file with no `.meta`. MetaGuard proposes creation of a new `.meta`. Policy action: Flag. |
| TC-3 | ZeroGUID | `TC3_ZeroGuidAsset.txt` + `.meta` with `guid: 000…000` | `.meta` contains all-zero GUID. MetaGuard proposes GUID regeneration. Policy action: AutoFix. |
| TC-4 | MalformedGUID | `TC4_MalformedGuidAsset.txt` + `.meta` with non-hex GUID | `.meta` contains an invalid GUID string. MetaGuard proposes GUID regeneration. Policy action: AutoFix. |
| TC-5 | BrokenReference | `TC5_BrokenRefAsset.asset` referencing GUID `deadbeef…dead` | YAML asset references a GUID that does not exist in the project. MetaGuard flags for review. Policy action: Flag. |

---

## How TC-1 and TC-2 Work

TC-1 and TC-2 require careful handling during setup because Unity's `AssetDatabase.Refresh()` call automatically repairs certain broken states — it will regenerate a missing `.meta` (TC-2) and log warnings about orphaned `.meta` files (TC-1) when it processes the test asset directory.

MetaGuard's setup tool writes TC-1 and TC-2 as the **last step** of setup, after all `AssetDatabase.Refresh()` calls have completed. Since no refresh follows, Unity has no opportunity to repair either test case. This ordering is what makes 5/5 READY states achievable.

If TC-1 or TC-2 show NOT READY after setup, you are likely running an older version of the package. Update to the latest version.

---

## Expected YAML Warnings (TC-3 and TC-4)

When Unity processes the test asset directory during setup, it emits warnings for TC-3 and TC-4:

```
The GUID inside 'TC3_ZeroGuidAsset.txt.meta' cannot be extracted by the YAML Parser.
The .meta file TC3_ZeroGuidAsset.txt.meta does not have a valid GUID...
```

These warnings are **expected and intentional**. They confirm the test cases are in the correct broken state. They are not MetaGuard errors.

---

## Validation

```
Tools > MetaGuard > Validate Demo State
```

Prints a report to the Console confirming which test cases are in the correct broken state:

```
[MetaGuard Demo] Validation Report
─────────────────────────────────────
TC-1 OrphanedMeta    : READY
TC-2 MissingMeta     : READY
TC-3 ZeroGUID        : READY
TC-4 MalformedGUID   : READY
TC-5 BrokenReference : READY
─────────────────────────────────────
Result: 5/5 test cases are in the expected broken state.
✓ All test cases ready — run 'Run Demo Scan' to validate MetaGuard.
```

If any case shows NOT READY, run **Setup Demo Assets** again.

---

## Running the Demo Scan

```
Tools > MetaGuard > Run Demo Scan
```

Opens the MetaGuard window and triggers **Scan + Analyze** automatically. After the scan completes, the Issues tab will show at minimum 5 issues across the five test case types.

Expected scan results with the default policy:

- **TC-1 OrphanedMeta** — detected as Low, AutoFix. Resolved by Fix All Safe.
- **TC-2 MissingMeta** — detected as Low, Flag. Requires manual Apply.
- **TC-3 ZeroGUID** — detected, AutoFix. Unity logs YAML warnings confirming the broken state.
- **TC-4 MalformedGUID** — detected, AutoFix. Unity logs YAML warnings confirming the broken state.
- **TC-5 BrokenReference** — detected, Flag. Requires manual Apply.

**Fix All Safe** will resolve TC-1, TC-3, and TC-4 automatically (AutoFix policy). TC-2 and TC-5 remain flagged for review.

---

## Teardown

```
Tools > MetaGuard > Teardown Demo Assets
```

Deletes all seeded test files from `Assets/MetaGuard/Demo/TestAssets/`. The demo scene, scripts, and assembly definitions are preserved. Run **Setup Demo Assets** again at any time to reseed.

---

## Demo Scene

`Assets/MetaGuard/Demo/MetaGuard_DemoScene.unity` contains:

- **DemoController** — `DemoBootstrap` MonoBehaviour that logs readiness on Awake and lists the seeded test cases
- **Main Camera** — Standard perspective camera with AudioListener attached
- **BrokenRefTestAnchor** — `DemoScriptReference` component; serves as the scene anchor for the TC-5 broken reference chain

The scene is written as raw YAML by the setup tool, bypassing `EditorSceneManager.SaveScene`, which avoids the "overwriting same open scene" assertion that fires when the target path is already loaded.

---

## Assembly Structure

The demo system uses a clean Editor/runtime split:

| Assembly | Scope | Contents |
|---|---|---|
| `MetaGuard.Demo` | Runtime | `DemoBootstrap`, `DemoDataConfig`, `DemoScriptReference` |
| `MetaGuard.Demo.Editor` | Editor-only | `DemoSetupEditor` — all menu items, test case seeding, scene writing |

`MetaGuard.Demo.Editor` references `MetaGuard.Demo`, `MetaGuard.Editor`, and `MetaGuard.Internal`. No runtime assembly references any Editor-only type.
