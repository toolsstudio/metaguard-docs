> ⚠️ **This is an archived version (v1). Use [MetaGuard Pro (v2)](../../versions/v2/) documentation instead.**

---

# MetaGuard 1.x Documentation

> **This is an archived version.** MetaGuard 1.x has reached end of life.
> For current documentation, return to the [main repository](../../README.md).

---

## Version Notice

MetaGuard 1.x was the initial release. It included the core scan, analysis, simulation, apply, and rollback pipeline. It did not include the policy system, CLI integration, health history, or demo system — those were introduced in 2.0.0.

---

## Documents

| Document | |
|---|---|
| [Getting Started](../../versions/v2/getting-started.md) | First scan walkthrough |
| [Features](../../versions/v2/features.md) | Feature reference |
| [Usage](../../versions/v2/usage.md) | UI reference |
| [Safety & Rollback](../../versions/v2/safety.md) | Snapshot and rollback model |
| [Cache System](../../versions/v2/cache-system.md) | Scan cache reference |
| [Troubleshooting](../../versions/v2/troubleshooting.md) | Common issues |
| [FAQ](../../versions/v2/faq.md) | Frequently asked questions |

---

## Upgrading to 2.x

MetaGuard 2.0.0 is a paid upgrade available on the Unity Asset Store.
It is published as a separate asset from the 1.x free version.

Key additions in 2.0.0:
- Policy system (`metaguard_policy.json`) — per-team rule enforcement
- CLI entry point for CI/CD pipeline integration
- Project health history log
- Demo system with five reproducible test cases
- Three-tier rollback with SHA-256 snapshot validation
- `excludeAssetPaths` for suppressing pipeline-managed asset false positives
- Scan depth defaulting to `AssetsAndPackages`

See the [2.x Changelog](../../CHANGELOG.md) for the complete list of changes.
