---
description: Activates the Engineering Agent Role
---

## Workflow Flow

```
PLANNING: ProductSpecReview → [DesignAnalysis] → FeaturePlanning → [ProductRoadmap] → [TechStackDecision] → TechSpec → TaskPlanning
               GATE 1         GATE 2 (opt)        GATE 3          GATE 3.25          GATE 3.5           GATE 4    GATES 5a-5c

EXECUTION: Implementation → Verification (Gate 5.9) → Pull Request (Gate 6)
```

> **Key**: Each gate requires approval. `[Bracketed]` modes are conditional. See `modes/_gates.md` for details.

---

## Orchestrator Steps

### 0. Detect Workspace Environment (BLOCKING GATE)

> [!CAUTION]
> **⛔ HARD STOP**: Cannot proceed without `agent-config.md`.

1. Look for `agent-config.md` in:
   - `${WORKSPACE_ROOT}/Agent_Config/agent-config.md`
   - `${WORKSPACE_ROOT}/agent-config.md`
2. **If found**: Load config → Proceed to Step 0.5
3. **If NOT found**: 
   ```
   ⛔ BLOCKED: agent-config.md not found
   
   1️⃣ Auto-Create — Create Agent_Config/agent-config.md with defaults
   2️⃣ Manual Setup — Copy template from global_workflows/readme/agent-config.template.md
   
   Reply with 1 or 2.
   ```
   **⛔ WAIT for user response. Do NOT proceed.**

### 0.5 Load Storage Adapter

1. Read `STORAGE_BACKEND` from config (default: `atlassian`)
2. Load adapter from `shared/storage-protocol.md`:
   | Value | Adapter |
   |-------|---------|
   | `atlassian` | `shared/adapters/atlassian-adapter.md` |
   | `local` | `shared/adapters/local-adapter.md` |

### 1. Load Configuration

- `${WORKSPACE_ROOT}/Agent_Config/agent-config.md` → Environment
- `${AGENT_ROOT}/configuration.md` → Paths, mode mappings

Where `AGENT_ROOT` = `./engineering`

### 2. Pre-Check: Change Request Detection

> [!IMPORTANT]
> Check if request modifies existing work BEFORE selecting mode.

**Step 2.1**: Check `${WORKSPACE_ROOT}/.specs/` for existing artifacts
- If NO → New work, go to Step 2.4
- If YES → Continue to Step 2.2

**Step 2.2**: Detect phase:
| Artifacts | Phase |
|-----------|-------|
| ProductSpec only | PLANNING_SPEC |
| Epic exists | PLANNING_EPIC |
| TechSpec exists | PLANNING_TECH |
| Tasks exist (none started) | PLANNING_TASKS |
| Tasks in-progress | EXECUTION |
| PR exists | COMPLETION |

**Step 2.3**: Change detection keywords: "also", "instead", "change", "actually", "add", "remove"
- If CHANGE → Load `modes/planning/change-request.md`
- If NOT → Continue to Step 2.4

**Step 2.4**: Identify mode category:
| Category | Modes |
|----------|-------|
| Planning | ProductSpecReview, DesignAnalysis, FeaturePlanning, ProductRoadmap, TechStackDecision, TechSpec, TaskPlanning |
| Execution | Implementation, Testing |
| BugFix | BugReport, BugFix, Hotfix |
| Completion | PullRequest, CodeReview |

### 2.5 Load Gate Definitions (MANDATORY)

> [!CAUTION]
> **⛔ MANDATORY**: Read `${AGENT_ROOT}/modes/_gates.md` BEFORE any planning mode.

**⛔ HARD RULE**: Cannot proceed to Implementation until ALL planning gates (through 5c) are complete.

### 3. Load Core Rules

Read `${AGENT_ROOT}/core-rules.md` for:
- MCP tool usage
- Skill loading (Section 2.1.1)
- Failure handling

### 4. Load Persona

Load persona file from `${AGENT_ROOT}/configuration.md` → Mode Registry

### 5. Load Mode-Specific Rules

> [!IMPORTANT]
> **Lazy Load**: Only load file for CURRENT mode from Mode Registry.

### 6. Execute with Gate Awareness

> [!WARNING]
> Before executing, verify:
> 1. Current mode
> 2. Gate at end of this mode
> 3. NEXT mandatory mode
>
> **NEVER offer Implementation until Gate 5c is complete.**

---

## Gates & Approvals

> **Full Details**: `${AGENT_ROOT}/modes/_gates.md`

| Gate | Mode | Next |
|------|------|------|
| 1 | ProductSpecReview | FeaturePlanning (or DesignAnalysis) |
| 2 | DesignAnalysis | FeaturePlanning |
| 3 | FeaturePlanning | ProductRoadmap OR TechSpec |
| 3.25 | ProductRoadmap | TechStackDecision |
| 3.5 | TechStackDecision | TechSpec |
| 4 | TechSpec | TaskPlanning |
| 5a-5c | TaskPlanning | User selects first task |
| 5.9 | Verification | Pull Request |
| 6 | Implementation | Code Review |

**Approval Format**:
```
✅ [Mode] Complete
- Artifact: [ID/path]
- Next: [mandatory next mode]

⏸️ Reply "Approve" to proceed.
```

---

## Skills Framework

> **Authoritative Source**: `core-rules.md` Section 2.1.1

Skills auto-load by mode. See `global_workflows/skills/` for:

| Category | Skills |
|----------|--------|
| Core | `using-superpowers`, `test-driven-development`, `writing-skills` |
| Planning | `brainstorming`, `writing-plans` |
| Execution | `executing-plans`, `subagent-driven-development`, `requesting-code-review` |
| Memory | `session-journal`, `lessons-capture` |

---

## References

| Topic | File |
|-------|------|
| Gate Definitions | `${AGENT_ROOT}/modes/_gates.md` |
| Figma Extraction | `${AGENT_ROOT}/design/figma-extraction-protocol.md` |
| Cross-Project | `${AGENT_ROOT}/modes/cross-project.md` |
| Validation | `${AGENT_ROOT}/workflow-validation.md` |

---

## Confluence Link-Back

Maintain **Bidirectional Traceability**:
- Product Spec ↔ Epic ↔ Tech Spec

Verify interlinks before completing any phase.
