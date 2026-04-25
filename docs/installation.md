# Installation

---

## Requirements

| Requirement | Details |
|---|---|
| Unity | 2020.3 LTS or later |
| Render Pipeline | Built-in, URP, and HDRP all supported |
| Platform | Windows, macOS, Linux (Editor only) |
| Runtime footprint | None — Editor-only assemblies, excluded from all builds |

---

## Importing the Package

1. Download the MetaGuard `.unitypackage` from the release page.
2. In the Unity menu bar, go to **Assets > Import Package > Custom Package…**
3. Select the downloaded `.unitypackage` file.
4. In the import dialog, confirm all items are checked and click **Import**.

MetaGuard installs entirely under `Assets/MetaGuard/`. Nothing is written outside this folder.

---

## Verifying the Installation

After import, the menu item **Tools > MetaGuard > Open MetaGuard** should appear in the Unity menu bar.

If it does not appear:

1. Check the Unity Console for compilation errors. MetaGuard has no external dependencies — any error indicates a file was not imported correctly.
2. Re-import the package. If errors persist, confirm the project meets the minimum Unity version requirement.
3. See [troubleshooting.md](troubleshooting.md) if the issue continues.

---

## Folder Layout After Import

```
Assets/
└── MetaGuard/
    ├── Editor/            ←  All editor tooling (Editor-only assembly)
    ├── Internal/          ←  Data model assembly
    ├── Docs/              ←  In-package documentation
    ├── Snapshots/         ←  Created on first Apply (not version-controlled)
    ├── scan_cache.txt     ←  Created on first scan with cache enabled
    └── CHANGELOG.md
```

All scripts are in Editor-only assemblies and are automatically excluded from runtime builds.

---

## Updating MetaGuard

1. Delete `Assets/MetaGuard/` from the project.
2. Import the new `.unitypackage` as described above.

> The `MetaGuard/Snapshots/` directory and `scan_cache.txt` are deleted along with the package. If you have a recent Apply session you may want to roll back first, or ensure the project is committed to source control before updating.

---

## Uninstalling

Delete `Assets/MetaGuard/` from the project. No other project files are modified by MetaGuard.

If you applied fixes and are satisfied with the result, no further action is needed before uninstalling.
