---
description: Activates the Engineering Agent Role
---

## Workflow Flow

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              PLANNING PHASE                                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│ ProductSpecReview → DesignAnalysis → FeaturePlanning → TechSpec → TaskPlanning  │
│       ⬇️               ⬇️              ⬇️             ⬇️           ⬇️            │
│   [APPROVE]       [APPROVE]      [APPROVE Epic]  [APPROVE Spec] [APPROVE Tasks] │
├─────────────────────────────────────────────────────────────────────────────────┤
│                            EXECUTION PHASE                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                      Implementation / BugFix                                     │
│                              ⬇️                                                  │
│                         [CODE COMPLETE]                                          │
├─────────────────────────────────────────────────────────────────────────────────┤
│                            COMPLETION PHASE                                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                         Pull Request                                             │
│                              ⬇️                                                  │
│                         [PR READY]                                               │
└─────────────────────────────────────────────────────────────────────────────────┘
```

> **Key Principle**: Each planning step requires human approval before proceeding.

---

## Orchestrator Steps

### 0. Detect Workspace Environment (MANDATORY)

> [!IMPORTANT]
> **Multi-Environment Support**: Agents load configuration from `${WORKSPACE_ROOT}/Agent_Config/agent-config.md`.

1. **Detect workspace root**: Use current working directory or VS Code workspace root
2. **Look for** `agent-config.md` in these locations (in order):
   - `${WORKSPACE_ROOT}/Agent_Config/agent-config.md`
   - `${WORKSPACE_ROOT}/agent-config.md`
3. **If found**: Load as primary configuration source
4. **If NOT found**: Prompt user to create one:
   ```
   ⚠️ **Workspace Configuration Required**
   
   No `agent-config.md` found in current workspace.
   
   Copy the template to your workspace:
   - Template: `global_workflows/readme/agent-config.template.md`
   - Destination: `${WORKSPACE_ROOT}/Agent_Config/agent-config.md`
   
   See setup instructions: `global_workflows/readme/setup_instructions.md`
   ```
5. **STOP** until `agent-config.md` exists

### 0.5 Load Storage Adapter (MANDATORY)

> [!NOTE]
> **Symmetric Architecture**: All backends load their adapter file.

1. **Read** `STORAGE_BACKEND` from `agent-config.md` (default: `atlassian`)
2. **Load adapter** from registry in `shared/storage-protocol.md`:
   | Value | Adapter |
   |-------|---------|
   | `atlassian` | `shared/adapters/atlassian-adapter.md` |
   | `local` | `shared/adapters/local-adapter.md` |
3. **If unknown value**: Error with valid options list

### 1. Load Configuration (MANDATORY)

Load from `${WORKSPACE_ROOT}/Agent_Config/agent-config.md` (detected above) + agent-specific config:
- `${WORKSPACE_ROOT}/agent-config.md` → Environment settings
- `${AGENT_ROOT}/configuration.md` → Agent paths, mode mappings

Where `AGENT_ROOT` = `./engineering`

### 2. Identify Mode

| Category | Modes |
|----------|-------|
| **Fast Track** | Small Jira Task (bypasses planning) |
| **Planning** | ProductSpecReview, DesignAnalysis, FeaturePlanning, TechSpec, TaskPlanning |
| **Execution** | Implementation, Testing |
| **BugFix** | BugReport, BugFix, Hotfix |
| **Completion** | PullRequest, CodeReview |

### 3. Load Core Rules

Read `${AGENT_ROOT}/core-rules.md` for MCP tool usage and failure handling.

### 4. Load Persona

> **Source**: See `${AGENT_ROOT}/configuration.md` → Mode Registry → Mode Mapping

Load the persona file corresponding to the current mode. For Execution/BugFix modes, use track-based persona selection.

### 5. Load Mode-Specific Rules

> [!IMPORTANT]
> **Lazy Loading**: Load ONLY the file for the CURRENT mode.
>
> **Source**: See `${AGENT_ROOT}/configuration.md` → Mode Registry → Mode Mapping

Look up the current mode in the Mode Registry and load its Rules File.

### 6. Execute

Proceed with the task using loaded context and mode-specific rules.

---

## Gates & Approvals

> **Full Details**: See `${AGENT_ROOT}/modes/_gates.md`

**Gate Checkpoints**:
1. After ProductSpecReview → Gap Analysis approved
2. After DesignAnalysis → Design Review approved (if Figma exists)
3. After FeaturePlanning → Epic approved
4. After TechSpec → Tech Spec approved
5. After TaskPlanning → 3 sequential gates (task list, scope, readiness)
6. After Implementation/BugFix → PR approved

**Standard Approval Format**:
```
✅ **[Mode Name] Complete**
- **Artifact**: [URL or Keys]
- **Summary**: [1-line description]
- **Next Step**: [Next mode]

> **⏸️ APPROVAL REQUIRED**: Reply with `Approve` to proceed.
```

---

## References

| Topic | File |
|-------|------|
| Figma Token Extraction | `${AGENT_ROOT}/design/figma-extraction-protocol.md` |
| Cross-Project Flow | `${AGENT_ROOT}/modes/cross-project.md` |
| Quality Validation | `${AGENT_ROOT}/workflow-validation.md` |
| Pre-Creation Validation | `${AGENT_ROOT}/modes/planning/_validation-checklist.md` |

---

## Confluence Link-Back (MANDATORY)

Maintain **Bidirectional Traceability**:

1. **Product Spec** → links to Epic and Tech Spec
2. **Jira Epic** → links to Product Spec and Tech Spec

Verify all 3 artifacts are interlinked before completing any phase.
