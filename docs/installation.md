# Installation

---

## Requirements

| Requirement | Details |
|---|---|
| Unity | 2020.3 LTS or later (including Unity 6) |
| Render Pipeline | Built-in, URP, and HDRP all supported |
| Platform | Windows, macOS, Linux (Editor only) |
| Runtime footprint | None — Editor-only assemblies, excluded from all builds |

---

## Purchasing and Downloading

MetaGuard 2.0.0 is available on the [Unity Asset Store](https://assetstore.unity.com/packages/slug/376206).

After purchase, download the `.unitypackage` through the Unity Package Manager or the Asset Store window inside the Unity Editor.

---

## Importing the Package

1. In the Unity menu bar, go to **Assets > Import Package > Custom Package…**
2. Select the downloaded `.unitypackage` file.
3. In the import dialog, confirm all items are checked and click **Import**.

MetaGuard installs entirely under `Assets/MetaGuard/`. Nothing is written outside this folder.

---

## Verifying the Installation

After import, the menu item **Tools > MetaGuard > Open MetaGuard** should appear in the Unity menu bar.

If it does not appear:

1. Open the Unity Console and check for compilation errors. MetaGuard has no external dependencies — any error indicates a file was not imported correctly.
2. Re-import the package via **Assets > Import Package > Custom Package…**. In the import dialog, confirm all items are checked before clicking Import.
3. Confirm the project meets the minimum Unity version requirement (2020.3 LTS or later).
4. See [troubleshooting.md](troubleshooting.md#the-metaguard-menu-item-does-not-appear) for additional steps.

---

## Folder Layout After Import

```
Assets/
└── MetaGuard/
    ├── Editor/              ←  All editor tooling (Editor-only assembly)
    ├── Internal/            ←  Data model assembly (runtime-safe, excluded from builds)
    ├── Demo/                ←  Demo scene and test case system
    ├── Snapshots/           ←  Created on first Apply (not version-controlled)
    ├── MetaGuardReports/    ←  Created on first CLI run (not version-controlled)
    ├── health_log.json      ←  Scan history log (optional: commit to track health over time)
    ├── metaguard_policy.json←  Policy rules (commit to share across the team)
    ├── scan_cache.txt       ←  Cache data (do not commit)
    └── CHANGELOG.md
```

All scripts are in Editor-only assemblies. They are excluded from runtime builds automatically — nothing from MetaGuard appears in a player build.

---

## Initial Policy Setup

On first use, MetaGuard creates a default `metaguard_policy.json` in `Assets/MetaGuard/`. This file controls how each issue class is handled — by the Editor tool and by CLI scans. The default configuration is:

| Issue Class | Default Action |
|---|---|
| GUIDCollision | Block |
| ZeroGUID | AutoFix |
| MalformedGUID | AutoFix |
| OrphanedMeta | AutoFix |
| MissingMeta | Flag |
| BrokenReference | Flag |
| CyclicDependency | Flag |

Commit `metaguard_policy.json` to source control so all team members and CI pipelines use the same rules.

→ See [policy.md](policy.md) for the full policy reference and action descriptions.

---

## Source Control Configuration

Add the following to your `.gitignore`:

```
Assets/MetaGuard/Snapshots/
Assets/MetaGuard/MetaGuardReports/
Assets/MetaGuard/scan_cache.txt
```

MetaGuard includes these entries in its own `.gitignore` by default. Verify they have not been removed.

**Commit** the following:
- `Assets/MetaGuard/metaguard_policy.json` — shared policy for team and CI
- `Assets/MetaGuard/health_log.json` — optional; enables team-wide health tracking over time

---

## Updating MetaGuard

1. Delete `Assets/MetaGuard/` from the project.
2. Import the new `.unitypackage` as described above.

> The `Snapshots/` directory and `scan_cache.txt` are deleted along with the package. If you have an active rollback session, roll back before deleting — or ensure the project is committed to source control first.

---

## Uninstalling

Delete `Assets/MetaGuard/` from the project. No other project files are modified by MetaGuard.

---

## Support

If installation fails or errors persist after following the steps above:

- [Discord](https://discord.gg/rYbZZz5GH4) — primary support channel
- [Bug reports](https://discord.gg/mQYguyhYwA) — for confirmed bugs
- See [troubleshooting.md](troubleshooting.md) for common issues and resolutions
