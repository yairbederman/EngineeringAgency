# Change Log Entry Template

> **Purpose**: Standard format for change log entries in artifacts.
> **Used by**: `storage.logChange()` in both local and Atlassian adapters.

---

## Entry Format (Markdown)

When appending to artifact markdown files:

```markdown
---
## Change Log

| Version | Date | Source | Type | Priority | Description |
|---------|------|--------|------|----------|-------------|
| v{version} | {date} | {source} | {type} | {priority} | {description} |
```

---

## Entry Format (JSON)

For `_registry.json` storage:

```json
{
  "version": "1.1",
  "date": "2026-01-11T22:00:00Z",
  "type": "increment",
  "source": "DEV",
  "priority": "P2",
  "description": "Added blog feature requirement",
  "impactedAreas": ["Epic", "TechSpec", "Tasks"],
  "codeImpact": {
    "filesAffected": ["src/components/Blog.tsx"],
    "tasksAffected": ["TASK-001"],
    "reworkEstimate": "2h"
  },
  "testImpact": {
    "testsModified": ["blog.test.tsx"],
    "testsAdded": ["blog-list.test.tsx"]
  },
  "approved": true,
  "approvedBy": "user"
}
```

---

## Entry Format (Jira Comment)

For Atlassian adapter `jira_add_comment`:

```markdown
🔄 **CHANGE LOG ENTRY**

- **Date**: {timestamp}
- **Source**: {source} (PM/DEV/STAKE/QA/TECH/EXT)
- **Priority**: {priority}
- **Type**: {type} (increment/adjustment/refinement)
- **Description**: {description}
- **Impact**: {impactedAreas}

---
*Logged by Engineering Agent*
```

---

## Entry Format (Confluence)

For Atlassian adapter `confluence_update_page`:

Append to "## Change Log" section:

```markdown
| v{version} | {date} | {source} | {type} | {priority} | {description} | {impactedAreas} |
```

---

## Field Definitions

### Source Codes

| Code | Meaning |
|------|---------|
| `PM` | Product Manager |
| `DEV` | Developer/User |
| `STAKE` | Stakeholder |
| `QA` | QA/Testing |
| `TECH` | Technical Debt |
| `EXT` | External (vendor, API) |

### Type Values

| Type | Definition |
|------|------------|
| `increment` | New feature/capability added |
| `adjustment` | Modification to existing scope |
| `refinement` | Clarification without scope change |
| `rollback` | Restored previous version |

### Priority Values

| Priority | Definition |
|----------|------------|
| `P0` | Blocker - Cannot ship without |
| `P1` | High - Significant value impact |
| `P2` | Medium - Enhances experience |
| `P3` | Low - Nice to have |

---

## Cascade Cross-Reference

When a change cascades to downstream artifacts, the entry should include:

```markdown
📎 Cascaded from {rootArtifactId}: {originalDescription}
```

Example:
```markdown
| v1.1 | 2026-01-11 | DEV | adjustment | P2 | 📎 Cascaded from SPEC-001: Added blog requirement |
```
