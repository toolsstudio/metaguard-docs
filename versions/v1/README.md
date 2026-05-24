# MetaGuard 1.x Documentation

> **This is an archived version.** MetaGuard 1.x has reached end of life.
> For current documentation, return to the [main repository](../../README.md).

---

> **Note:** These archived docs were written alongside 2.x development and contain forward-references to features introduced in 2.x (policy system, CLI integration, demo system). Those sections describe 2.x behaviour, not 1.x. Treat this archive as a historical reference only.

---

## Version Notice

MetaGuard 1.x was the initial release. It included the core scan, analysis, simulation, apply, and rollback pipeline. It did not include the policy system, CLI integration, health history, or demo system — those were introduced in 2.x.

---

## Documents

| Document | |
|---|---|
| [Getting Started](docs/getting-started.md) | First scan walkthrough |
| [Features](docs/features.md) | Feature reference |
| [Usage](docs/usage.md) | UI reference |
| [Safety & Rollback](docs/safety.md) | Snapshot and rollback model |
| [Cache System](docs/cache-system.md) | Scan cache reference |
| [Troubleshooting](docs/troubleshooting.md) | Common issues |
| [FAQ](docs/faq.md) | Frequently asked questions |

---

## Upgrading to 2.x

MetaGuard 2.x is a paid upgrade available on the Unity Asset Store.
It is published as a separate asset from the 1.x free version.

Key additions in 2.x:
- Policy system (`metaguard_policy.json`) — per-team rule enforcement
- CLI entry point for CI/CD pipeline integration
- Project health history log
- Demo system with five reproducible test cases
- Three-tier rollback with SHA-256 snapshot validation
- `excludeAssetPaths` for suppressing pipeline-managed asset false positives
- Scan depth defaulting to `AssetsAndPackages`

See the [2.x Changelog](../../CHANGELOG.md) for the complete list of changes.
