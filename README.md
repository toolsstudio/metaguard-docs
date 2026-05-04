<div align="center">

<<<<<<< HEAD
<img src="images/MG_Cover.png" alt="MetaGuard Pro — Unity GUID Integrity Engine" width="100%">
=======
<img src="images/MG_Cover.png" alt="MetaGuard — Unity GUID Integrity Engine" width="100%">
>>>>>>> 535bab848ab29badabd4510c97f00570ec3ca11a

<br><br>

[![Unity](https://img.shields.io/badge/Unity-2020.3%2B-black?logo=unity&logoColor=white)](https://unity.com)
[![Version](https://img.shields.io/badge/Version-2.0.0-brightgreen)](CHANGELOG.md)
[![Platform](https://img.shields.io/badge/Platform-Editor%20Only-blue)]()
[![Asset Store](https://img.shields.io/badge/Asset%20Store-376206-orange)](https://assetstore.unity.com/packages/slug/376206)

</div>

<<<<<<< HEAD
> **This documentation reflects MetaGuard Pro 2.x (current stable).** For older versions, see [Legacy Version](#legacy-version) below.
=======
---

## Overview

Unity projects accumulate GUID corruption silently. Broken asset references, duplicate GUIDs, and orphaned `.meta` files compound over time — surfacing during merges, after directory restructuring, or before release, when the cost of fixing them is highest.

MetaGuard detects every class of GUID integrity issue across an entire project, simulates every proposed fix against a live dependency graph, and applies only safe operations — with a full pre-apply snapshot and 48-hour rollback guarantee.

In 2.0.0, MetaGuard adds a policy system for per-team rule enforcement, a CLI entry point for CI/CD integration, a project health history log, and a complete demo system for validation.

> **No file is modified without a recoverable snapshot.**
> **No operation reaches Apply without passing simulation.**

---

## Features

| Feature | Description |
|---|---|
| 🔍 **Full Project Scan** | Enumerates all `.meta` files, extracts GUIDs, maps every asset relationship. Defaults to Assets + Packages depth to resolve pipeline-managed references. |
| 🕸️ **Dependency Graph** | Builds a directed reference graph for accurate pre-fix impact analysis |
| 🧩 **7-Class Issue Detection** | GUID collisions, zero GUIDs, malformed GUIDs, orphaned/missing metas, broken references, cyclic dependencies |
| 📊 **Risk Classification** | Every issue rated Critical / High / Medium / Low |
| 🧪 **Simulation Engine** | Every fix tested against an in-memory graph clone — no files touched |
| ✅ **Safe Apply Pipeline** | Atomic writes, staged through temp files, Unity asset database batched |
| 📋 **Policy System** | Per-project `metaguard_policy.json` — controls issue handling across the team and in CI |
| 🔁 **Auto-Fix Controller** | GUID regeneration, orphan deletion, missing meta creation — one click |
| ⚡ **Scan Cache** | Skips unchanged files on repeated scans for faster iteration |
| 🛡️ **Three-Tier Rollback** | Snapshot → persistent session → automatic failure recovery |
| 🖥️ **CLI / CI Integration** | Headless batch-mode entry point, JSON reports, deterministic exit codes |
| 📈 **Health History** | Per-scan health score log committed to source control |

---

## Workflow

```
Scan  ──►  Analyze  ──►  Simulate  ──►  Apply  ──►  Rollback (if needed)
```

1. **Scan** — Enumerate all assets, extract GUIDs, build the dependency graph.
2. **Analyze** — Detect issues, classify risk, report a project health score (0–100).
3. **Simulate** — Test every proposed fix. See the verdict before anything is written.
4. **Apply** — Write only simulation-approved operations, protected by a pre-apply snapshot.
5. **Rollback** — Restore all modified files from the snapshot in a single action.
>>>>>>> 535bab848ab29badabd4510c97f00570ec3ca11a

---

## Quick Start

```
<<<<<<< HEAD
Tools > MetaGuard Pro > Open
=======
Tools > MetaGuard > Open MetaGuard
>>>>>>> 535bab848ab29badabd4510c97f00570ec3ca11a
```

1. Click **Scan + Analyze**
2. Review detected issues in the **Issues** tab
3. Click **Simulate** — confirm all intended operations show **Safe**
4. Click **Apply** or **Fix All Safe**
5. If the result is unexpected — click **Rollback**

<<<<<<< HEAD
---

## Overview

Unity projects accumulate GUID corruption silently. Broken asset references, duplicate GUIDs, and orphaned `.meta` files compound over time — surfacing during merges, after directory restructuring, or at release.

MetaGuard Pro scans every asset in the project, builds a directed dependency graph, classifies every GUID integrity issue by risk level, tests every proposed fix against the graph before touching any file, and applies only operations it can verify are safe — with a pre-apply snapshot and 48-hour rollback window on every session.

> No file is modified without a recoverable snapshot.
> No operation reaches Apply without passing simulation.

---

## Pipeline

```
Scan  ──►  Analyze  ──►  Simulate  ──►  Apply  ──►  Rollback
```
=======
→ See [Getting Started](docs/getting-started.md) for the full walkthrough.
>>>>>>> 535bab848ab29badabd4510c97f00570ec3ca11a

---

## Requirements

<<<<<<< HEAD
- Unity 2020.3 LTS or later (including Unity 6)
- All render pipelines: Built-in, URP, HDRP
- Editor-only — zero runtime footprint, excluded from all builds
=======
- **Unity 2020.3 LTS** or later (including Unity 6)
- All render pipelines supported (Built-in, URP, HDRP)
- Editor-only — zero runtime footprint, not included in builds
>>>>>>> 535bab848ab29badabd4510c97f00570ec3ca11a

---

## Documentation

| Document | Description |
|---|---|
<<<<<<< HEAD
| [Getting Started](versions/v2/getting-started.md) | First scan, first rollback, recommended workflow |
| [Installation](versions/v2/installation.md) | Import steps, folder layout, updating, uninstalling |
| [Usage](versions/v2/usage.md) | Every button, tab, and control explained |
| [Features](versions/v2/features.md) | Full feature reference |
| [Policy System](versions/v2/policy.md) | Per-project rules, team sharing, CI enforcement |
| [CLI & CI Integration](versions/v2/cli.md) | Batch mode, JSON reports, exit codes, GitHub Actions |
| [Safety & Rollback](versions/v2/safety.md) | Snapshot model, session lifecycle, crash recovery |
| [Cache System](versions/v2/cache-system.md) | When to enable, disable, and reset the scan cache |
| [Demo System](versions/v2/demo.md) | Seeding and validating the test environment |
| [Best Practices](versions/v2/best-practices.md) | Team workflows, source control, CI gates |
| [Troubleshooting](versions/v2/troubleshooting.md) | Common issues and resolutions |
| [FAQ](versions/v2/faq.md) | Frequently asked questions |
=======
| [Getting Started](docs/getting-started.md) | First scan, first rollback, recommended workflow |
| [Installation](docs/installation.md) | Import steps, folder layout, updating, uninstalling |
| [Usage](docs/usage.md) | Every button, tab, and control explained |
| [Features](docs/features.md) | Full feature reference |
| [Policy System](docs/policy.md) | Per-project rules, team sharing, CI enforcement |
| [CLI & CI Integration](docs/cli.md) | Batch mode, JSON reports, exit codes, GitHub Actions example |
| [Safety & Rollback](docs/safety.md) | Snapshot and rollback model in detail |
| [Cache System](docs/cache-system.md) | When to enable, disable, and reset the scan cache |
| [Demo System](docs/demo.md) | Seeding and validating the test environment |
| [Best Practices](docs/best-practices.md) | Team workflows, source control, CI gates |
| [Troubleshooting](docs/troubleshooting.md) | Common issues and their resolutions |
| [FAQ](docs/faq.md) | Frequently asked questions |
>>>>>>> 535bab848ab29badabd4510c97f00570ec3ca11a
| [Changelog](CHANGELOG.md) | Full version history |

---

<<<<<<< HEAD
=======
## Repository Structure

```
MetaGuard-Docs/
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── documentation_request.md
│   └── PULL_REQUEST_TEMPLATE.md
├── docs/
│   ├── getting-started.md
│   ├── installation.md
│   ├── usage.md
│   ├── features.md
│   ├── policy.md
│   ├── cli.md
│   ├── safety.md
│   ├── cache-system.md
│   ├── demo.md
│   ├── best-practices.md
│   ├── troubleshooting.md
│   └── faq.md
├── images/
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── SECURITY.md
└── SUPPORT.md
```

---

>>>>>>> 535bab848ab29badabd4510c97f00570ec3ca11a
## Support

| Channel | |
|---|---|
<<<<<<< HEAD
| **Discord** | [discord.gg/rYbZZz5GH4](https://discord.gg/rYbZZz5GH4) — primary support channel |
| **Bug Reports** | [discord.gg/mQYguyhYwA](https://discord.gg/mQYguyhYwA) |
| **Email** | tools.studio@zohomail.in |
| **Asset Store** | [assetstore.unity.com/packages/slug/376206](https://assetstore.unity.com/packages/slug/376206) |

---

## Legacy Version

MetaGuard 1.x (free, end-of-life) documentation is preserved at [versions/v1/](versions/v1/).
=======
| Discord | [Join Discord](https://discord.gg/rYbZZz5GH4) — primary support channel |
| Bug Reports | [GitHub Issues](https://github.com/Afterix-Hub/MetaGuard) |
| Email | [Contact via Email](mailto:tools.studio@zohomail.in) |
| Asset Store | [View on Unity Asset Store](https://assetstore.unity.com/packages/slug/376206) |

See [SUPPORT.md](SUPPORT.md) for full details.

---

## Contributing

Documentation improvements, corrections, and additions are welcome.
See [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.
>>>>>>> 535bab848ab29badabd4510c97f00570ec3ca11a

---

<div align="center">

**Tools Studio** — Professional Unity Tooling

</div>
