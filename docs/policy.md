# Policy System

MetaGuard's policy system controls how each issue class is handled — in the Editor tool, during Fix All Safe, and in CLI scans. A single JSON file governs all three contexts, ensuring consistent enforcement across developer machines and CI pipelines.

---

## The Policy File

Location: `Assets/MetaGuard/metaguard_policy.json`

MetaGuard creates this file with default values when the window is opened for the first time. To create it manually:

```
Tools > MetaGuard > Create Default Policy File
```

Commit the file to source control. Every team member and every CI run will use the same rules.

```json
{
  "version": 1,
  "description": "MetaGuard Fix Policy v2.0 — commit this file to share policy across the team. Same file used by CLI for CI/CD scans.",
  "rules": {
    "GUIDCollision":    "Block",
    "ZeroGUID":         "AutoFix",
    "MalformedGUID":    "AutoFix",
    "OrphanedMeta":     "AutoFix",
    "MissingMeta":      "Flag",
    "BrokenReference":  "Flag",
    "CyclicDependency": "Flag"
  },
  "excludeAssetPaths": [
    "Assets/DefaultVolumeProfile.asset",
    "Assets/UniversalRenderPipelineGlobalSettings.asset"
  ]
}
```

---

## Actions

Each issue class maps to exactly one action.

### AutoFix

MetaGuard generates a fix suggestion and marks it auto-eligible. Fix All Safe and the CLI apply AutoFix operations without requiring user confirmation (unless the simulation verdict is not Safe).

Use AutoFix for issue classes where the correct fix is unambiguous and the risk of applying without review is low. ZeroGUID, MalformedGUID, and OrphanedMeta are the appropriate defaults — the correct fix for each is deterministic.

### Flag

MetaGuard generates a fix suggestion and surfaces it in the Suggestions tab, but does not apply it automatically. The user must review and apply it explicitly via Apply or Fix Selected.

Use Flag for issue classes where the correct action depends on project context. BrokenReference and MissingMeta are the defaults because missing or unreferenced assets may reflect intentional deletion rather than corruption.

### Ignore

MetaGuard detects the issue but does not surface it in the Issues tab, does not generate a suggestion, and does not include it in the health score calculation. Ignored issues also do not appear in CLI reports.

Use Ignore sparingly. Ignored issues are completely invisible to the user and will not appear in any output.

### Block

In the Editor: treated identically to Flag — the issue is surfaced and a suggestion is generated.

In CLI scans: sets `has_violations = true` and ensures exit code 1, regardless of issue counts or threshold. Use Block for issue classes that must never pass a CI gate.

GUIDCollision is set to Block by default because two assets sharing a GUID causes non-deterministic asset loading — a hard correctness failure that should always break a CI pipeline.

---

## Violation Logic (CLI)

A CLI scan sets `has_violations: true` (exit code 1) when **any** of the following are true:

- `critical > 0` — at least one Critical-risk issue
- `high > 0` — at least one High-risk issue
- `health_score == 0` — always a violation regardless of other counts
- `health_score <= threshold` — health score at or below the configured `--threshold` value
- A `Block` policy rule matches any detected issue

All five conditions are evaluated independently. A project with no Critical or High issues but a Block-matched issue will still exit with code 1.

The following debug line is emitted to both stderr and the Unity `-logFile` after every evaluation:

```
ViolationCheck => critical:0 high:0 threshold:4 result:false
```

This is emitted regardless of the result and is useful for diagnosing unexpected pass/fail outcomes in CI.

---

## excludeAssetPaths

Paths listed in `excludeAssetPaths` are excluded from BrokenReference analysis. Issues detected in these assets are not surfaced, not included in suggestions, and not counted in the health score.

This field exists to handle pipeline-managed assets. Unity's URP `UniversalRenderPipelineGlobalSettings.asset` and `DefaultVolumeProfile.asset` reference GUIDs in the `Packages/` folder. These references are valid at runtime but may appear broken to a scanner operating at the filesystem level, particularly on machines where certain package versions are not installed. Excluding these paths suppresses the false positives without affecting BrokenReference detection anywhere else in the project.

The default policy excludes:
```json
"excludeAssetPaths": [
  "Assets/DefaultVolumeProfile.asset",
  "Assets/UniversalRenderPipelineGlobalSettings.asset"
]
```

Add additional paths using the same project-relative format (`Assets/...`). All comparisons are case-insensitive on Windows and case-sensitive on macOS and Linux.

---

## Reloading the Policy

The policy is loaded when the MetaGuard window opens and when you click **Reload** in the Policy tab. To apply changes made to the JSON file:

1. Edit `Assets/MetaGuard/metaguard_policy.json` directly
2. Open the **Policy** tab in the MetaGuard window
3. Click **Reload**

The next scan will use the updated rules. The CLI always reads the policy file fresh at the start of each run — no reload step is required in CI.

---

## Policy Tab

The Policy tab in the MetaGuard window shows:

- The loaded file path and a reminder to commit it to source control
- The full rules table: issue class, action, and description
- The Action Legend: what each action means in plain language
- A note confirming the same file is used by the CLI

---

## Team Workflows

### Shared policy, no overrides

Commit `metaguard_policy.json` to source control. All team members and all CI runs use the same file. This is the standard configuration.

### Environment-specific thresholds

The `--threshold` CLI argument overrides the health score threshold for a specific run without modifying the policy file. This allows a development pipeline to tolerate a lower score while a release gate enforces a higher one.

```bash
# Development pipeline — tolerate down to 85
-- --threshold 3

# Release gate — fail below 98
-- --threshold 1
```

### Promoting Flagged issues to AutoFix

Once the team is confident that a particular issue class is always safe to auto-fix in your project, change its action from `Flag` to `AutoFix`. Fix All Safe and the CLI will then apply it automatically on every run.

### Suppressing known false positives

If a specific asset is producing BrokenReference false positives that cannot be resolved — for example, a third-party plugin with an unusual GUID structure — add its path to `excludeAssetPaths` rather than setting `BrokenReference` to `Ignore`. Path-level exclusion is specific and reversible. `Ignore` suppresses detection globally and permanently until the policy is changed.

---

## Default Policy Rationale

| Issue | Default | Reason |
|---|---|---|
| GUIDCollision | Block | Two assets sharing a GUID causes non-deterministic asset loading. Always a hard CI gate failure. |
| ZeroGUID | AutoFix | All-zero GUIDs are always wrong. Regeneration with a new unique GUID is the only correct fix. |
| MalformedGUID | AutoFix | Invalid GUID format. Same rationale as ZeroGUID — no ambiguity about the correct fix. |
| OrphanedMeta | AutoFix | A `.meta` with no asset is always a leftover from an incomplete deletion. Deletion is safe. |
| MissingMeta | Flag | May indicate intentional deletion outside of Unity. Requires human review before acting. |
| BrokenReference | Flag | The referenced asset may have been intentionally removed. Context required before acting. |
| CyclicDependency | Flag | Cycles are not always errors — some asset architectures deliberately use them. Review required. |
