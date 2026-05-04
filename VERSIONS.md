# Version Index

This file is the stable entry point for all version-specific documentation.
All paths below are permanent and will not change when new versions are released.

| Version | Status | Documentation |
|---|---|---|
| **2.x** | Current stable | [README.md](README.md) and [versions/v2/](versions/v2/) |
| 1.x | End of life | [versions/v1/](versions/v1/) |

---

## Version Lifecycle

Documentation for a version is maintained in the repository root (`/docs/`) while that version is current stable.

When a new major version is released:
1. The current `/versions/v2/` contents are archived to `/versions/vN/`
2. New content is written to `/versions/v2/`
3. The root `README.md` is updated with the new version notice
4. The archived version entry is added to this file

All version-specific paths remain stable indefinitely after archiving.

---

## Linking to Documentation

For external links that must remain stable across releases — such as the Unity Asset Store page, in-product help buttons, or CI pipeline documentation links — always use:

```
https://github.com/toolsstudio/metaguard-docs
```

This URL resolves to the current stable documentation at all times.

Do not link directly to individual documents or specific version paths in contexts that may outlive the current version (e.g., in-product URLs, marketplace listings, CI configuration examples).
