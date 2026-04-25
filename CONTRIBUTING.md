# Contributing to MetaGuard Docs

Thank you for taking the time to improve the MetaGuard documentation.

This repository contains **documentation only**. The MetaGuard source code is publicly available at [GitHub](https://github.com/Afterix-Hub/MetaGuard).

---

## What Belongs Here

- Corrections to existing documentation (typos, inaccurate descriptions, outdated steps)
- Clarity improvements to existing content
- New troubleshooting entries for confirmed, reproducible issues
- New FAQ entries for questions that arise frequently
- Additional usage examples

## What Does Not Belong Here

- Source code of any kind
- Internal implementation details, class names, or architecture descriptions
- Debug logs or dev notes
- Content that exposes non-public API surface

---

## How to Contribute

1. **Fork** this repository.
2. **Create a branch** named after the change: `fix/installation-typo`, `docs/add-faq-entry`, etc.
3. **Make your changes** following the style guidelines below.
4. **Open a pull request** with a clear description of what changed and why.

---

## Style Guidelines

**Tone**
- Write for a developer audience. Assume Unity familiarity, not MetaGuard familiarity.
- Direct and factual. No filler, no marketing language.
- Present tense: "MetaGuard creates a snapshot" not "MetaGuard will create a snapshot."

**Formatting**
- Use ATX headings (`##`, `###`) — no underline-style headings.
- Use tables for structured comparisons, not bullet lists.
- Use fenced code blocks (` ``` `) for paths, file names, and menu navigation sequences.
- One blank line between sections. Two blank lines before a top-level `##` heading.
- Filenames in code formatting: `scan_cache.txt`, `MetaGuard/Snapshots/`.
- UI elements in bold: **Scan + Analyze**, **Fix All Safe**, **Rollback**.

**File Naming**
- Lowercase, hyphen-separated: `cache-system.md`, `getting-started.md`.
- No spaces in file or folder names.

---

## Pull Request Checklist

Before submitting, confirm:

- [ ] No source code or internal class names included
- [ ] No new files added outside `docs/`, `images/`, or repo root
- [ ] Style guidelines followed
- [ ] All internal links checked and working
- [ ] Spelling and grammar reviewed

---

## Reporting Issues

Use the issue templates in `.github/ISSUE_TEMPLATE/`:

- **Bug report** — for incorrect or misleading documentation
- **Documentation request** — for missing content or coverage gaps

---

## Questions

Open a [GitHub Discussion](https://github.com/Afterix-Hub/MetaGuard/discussions) or join the [Discord](https://discord.gg/QNSJZGvRYM) support channel.
