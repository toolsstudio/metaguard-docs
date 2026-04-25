<div align="center">

# MetaGuard

**GUID Integrity System for Unity**

[![Unity](https://img.shields.io/badge/Unity-2020.3%2B-black?logo=unity)](https://unity.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Editor%20Only-blue)]()
[![Version](https://img.shields.io/badge/Version-1.0.0-orange)]()

*Scan. Simulate. Apply. Roll back if needed.*

</div>

---

## Overview

Unity projects accumulate GUID corruption silently. Broken asset references, duplicate GUIDs, and orphaned `.meta` files compound over time — surfacing during merges, after directory restructuring, or before Asset Store submissions, when the cost of fixing them is highest.

MetaGuard detects every class of GUID integrity issue across an entire project, simulates every proposed fix against a live dependency graph, and applies only safe operations — with a full pre-apply snapshot and 48-hour rollback guarantee.

> **No file is modified without a recoverable snapshot.**  
> **No operation is applied without a simulation verdict.**

---

## Features

| Feature | Description |
|---|---|
| 🔍 **Full Project Scan** | Enumerates all `.meta` files, extracts GUIDs, maps every asset relationship |
| 🕸️ **Dependency Graph** | Builds a directed reference graph for accurate pre-fix impact analysis |
| 🧩 **7-Class Issue Detection** | GUID collisions, zero GUIDs, malformed GUIDs, orphaned/missing metas, broken references, cyclic dependencies |
| 📊 **Risk Classification** | Every issue rated Critical / High / Medium / Low |
| 🧪 **Simulation Engine** | Every fix tested against an in-memory graph clone — no files touched |
| ✅ **Safe Apply Pipeline** | Atomic writes, staged through temp files, Unity asset database batched |
| 🔁 **Auto-Fix Controller** | GUID regeneration, orphan deletion, missing meta creation — one click |
| ⚡ **Scan Cache** | Skips unchanged files on repeated scans for faster iteration |
| 🛡️ **Three-Tier Rollback** | Snapshot → persistent session → automatic failure recovery |

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

---

## Quick Start

```
Tools > MetaGuard > Open MetaGuard
```

1. Click **Scan + Analyze**
2. Review detected issues in the **Issues** tab
3. Click **Simulate** — confirm all intended operations show **Safe**
4. Click **Apply** or **Fix All Safe**
5. If the result is unexpected — click **Rollback**

→ See [docs/getting-started.md](docs/getting-started.md) for the full walkthrough.

---

## Requirements

- **Unity 2020.3 LTS** or later
- All render pipelines supported
- Editor-only — zero runtime footprint, not included in builds

---

## Documentation

| Document | Description |
|---|---|
| [Getting Started](docs/getting-started.md) | Installation, first scan, first rollback |
| [Installation](docs/installation.md) | Import steps, folder layout, updating, uninstalling |
| [Usage](docs/usage.md) | Every button, tab, and control explained |
| [Features](docs/features.md) | Full feature reference |
| [Safety](docs/safety.md) | Snapshot and rollback model in detail |
| [Cache System](docs/cache-system.md) | When to enable, disable, and reset the scan cache |
| [Troubleshooting](docs/troubleshooting.md) | Common issues and their resolutions |
| [FAQ](docs/faq.md) | Frequently asked questions |

---

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
│   ├── safety.md
│   ├── cache-system.md
│   ├── troubleshooting.md
│   └── faq.md
├── images/
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
└── SECURITY.md
```

---

## Contributing

Documentation improvements, corrections, and additions are welcome.  
See [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

---

## License

MIT — see [LICENSE](LICENSE) for details.

---

<div align="center">

**Tools Studio** — Professional Unity Tooling

</div>
