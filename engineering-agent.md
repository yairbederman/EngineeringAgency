---
description: Activates the Engineering Agent Role
---

## Workflow Flow

```
┌───────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                        PLANNING PHASE                                              │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                    │
│  ProductSpecReview → [DesignAnalysis] → FeaturePlanning → [ProductRoadmap] → [TechStackDecision] │
│        ⬇️                  ⬇️                ⬇️                 ⬇️                  ⬇️             │
│    [APPROVE]          [APPROVE]        [APPROVE Epic]    [APPROVE Roadmap]   [APPROVE Stack]     │
│                       (if Figma)                         (if new project)    (if new project)    │
│                                                                                                    │
│                                  → TechSpec → TaskPlanning                                         │
│                                       ⬇️           ⬇️                                              │
│                                  [APPROVE Spec] [APPROVE Tasks]                                    │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                        EXECUTION PHASE                                             │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                    Implementation / BugFix                                         │
│                                            ⬇️                                                      │
│                                       [CODE COMPLETE]                                              │
│                                            ⬇️                                                      │
│                         🔍 Gate 5.9: Live Verification Gate                                        │
│                          ├── DOC_ONLY ────────► Auto-approve                                      │
│                          ├── BUILD_CHECK ─────► Build pass required                               │
│                          ├── API_TEST ────────► Contract validation required                      │
│                          └── BROWSER_VISUAL ──► User approval required                            │
│                                            ⬇️                                                      │
│                                  [VERIFICATION APPROVED]                                           │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                       COMPLETION PHASE                                             │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                  🔍 Gate 6: Pull Request                                           │
│                                            ⬇️                                                      │
│                                        [PR READY]                                                  │
└───────────────────────────────────────────────────────────────────────────────────────────────────┘
```

> **Key Principle**: Each planning step requires human approval before proceeding.
> **Conditional Steps**: `[DesignAnalysis]`, `[ProductRoadmap]`, and `[TechStackDecision]` are conditional—see `modes/_gates.md` for trigger conditions.
> **Gate 5.9**: Verification MUST pass before Gate 6 (PR) can begin.

---

## Orchestrator Steps

### 0. Detect Workspace Environment (BLOCKING GATE)

> [!CAUTION]
> **⛔ HARD STOP: This step is NON-NEGOTIABLE.**
> 
> The agent **CANNOT** proceed to ANY other step without a valid `agent-config.md`.
> **DO NOT** attempt to help the user, analyze code, or do ANY work until this gate passes.
> **DO NOT** offer workarounds or alternatives. The config file is MANDATORY.

1. **Detect workspace root**: Use current working directory or VS Code workspace root
2. **Look for** `agent-config.md` in these locations (in order):
   - `${WORKSPACE_ROOT}/Agent_Config/agent-config.md`
   - `${WORKSPACE_ROOT}/agent-config.md`
3. **If found**: Load as primary configuration source → Proceed to Step 0.5
4. **If NOT found**: 

   > [!WARNING]
   > **⛔ WORKFLOW BLOCKED: Missing Configuration**

   **IMMEDIATELY stop all work and present ONLY this message:**

   ```
   ⛔ **BLOCKED: agent-config.md not found**
   
   I cannot proceed without a workspace configuration file.
   
   **Choose an option:**
   
   1️⃣ **Auto-Create** — I'll create `Agent_Config/agent-config.md` with defaults.
      - Storage: `local` (file-based, no Atlassian required)
      - You can customize it afterward.
   
   2️⃣ **Manual Setup** — Copy the template yourself:
      - Template: `global_workflows/readme/agent-config.template.md`
      - Destination: `${WORKSPACE_ROOT}/Agent_Config/agent-config.md`
   
   Reply with `1` or `2` to proceed.
   ```

   **⛔ DO NOT proceed to Step 0.5 or any other step. WAIT for user response.**

5. **If user selects `1` (Auto-Create)**:
   - Create directory: `${WORKSPACE_ROOT}/Agent_Config/`
   - Copy `global_workflows/readme/agent-config.template.md` to `${WORKSPACE_ROOT}/Agent_Config/agent-config.md`
   - Notify user: "✅ Created `Agent_Config/agent-config.md`. Review and customize values, then **re-invoke the agent**."
   - **⛔ HARD STOP** — Do NOT continue. Agent must be re-invoked after config review.

6. **If user selects `2` (Manual Setup)**:
   - **⛔ HARD STOP** — Wait until user confirms config exists, then re-invoke agent.

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

### 2. Pre-Check: Is This a Change Request?

> [!IMPORTANT]
> **Before selecting a mode**, check if this is a modification to existing work.

#### Step 2.1: Active Artifact Check

```
Check: Does ${WORKSPACE_ROOT}/.specs/ contain existing artifacts?

If NO → New work, proceed to Step 2.4 (Identify Mode)
If YES → Continue to Step 2.2
```

#### Step 2.2: Phase Detection

Determine the current workflow phase based on existing artifacts:

| Artifacts Exist | Task Status | Phase |
|-----------------|-------------|-------|
| ProductSpec only | N/A | PLANNING_SPEC |
| Epic exists | No tasks | PLANNING_EPIC |
| TechSpec exists | No tasks | PLANNING_TECH |
| Tasks exist | None started | PLANNING_TASKS |
| Tasks exist | Some in-progress | EXECUTION |
| PR mentioned/exists | - | COMPLETION |

#### Step 2.3: Change Detection

Does the user's request modify existing scope?

**Change Indicators**:
- Keywords: "also...", "instead...", "change...", "actually...", "add...", "remove..."
- References existing feature with modification intent
- Introduces new requirement after spec approval

**NOT a Change**:
- User says "start fresh" or "new project"
- Request is about a different, unrelated feature
- Request is "continue with [existing task]"

```
If CHANGE DETECTED:
  1. Load ${AGENT_ROOT}/modes/planning/change-request.md
  2. Execute ChangeRequest protocol
  3. After change applied → Resume at current gate
  
If NOT A CHANGE:
  Continue to Step 2.4 (Identify Mode)
```

#### Step 2.4: Identify Mode

| Category | Modes |
|----------|-------|
| **Change Request** | ChangeRequest (mid-flow modifications) |
| **Fast Track** | Small Jira Task (bypasses planning) |
| **Planning** | ProductSpecReview, DesignAnalysis, FeaturePlanning, ProductRoadmap, TechStackDecision, TechSpec, TaskPlanning |
| **Execution** | Implementation, Testing |
| **BugFix** | BugReport, BugFix, Hotfix |
| **Completion** | PullRequest, CodeReview |

### 2.5 Load Gate Definitions (MANDATORY)

> [!CAUTION]
> **⛔ MANDATORY: This step is NON-NEGOTIABLE for ALL Planning modes.**
>
> You MUST read `${AGENT_ROOT}/modes/_gates.md` BEFORE executing ANY planning mode.
> This file defines the **complete planning chain** and all gate requirements.

**Load and internalize the Planning Chain**:

```
ProductSpecReview → [DesignAnalysis] → FeaturePlanning → [ProductRoadmap] → [TechStackDecision] → TechSpec → TaskPlanning → Implementation → Verification → PR
     GATE 1          GATE 2 (opt)        GATE 3          GATE 3.25          GATE 3.5           GATE 4    GATES 5a-5c      EXECUTION      GATE 5.9    GATE 6
```

**⛔ HARD RULE**: You **CANNOT** proceed to Implementation until ALL applicable planning gates (through Gate 5c) are complete.

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

### 6. Execute with Gate Awareness

> [!WARNING]
> **Before executing, verify your position in the planning chain.**
>
> Ask yourself:
> 1. What is my current mode?
> 2. What gate will I reach at the end of this mode?
> 3. What is the NEXT mandatory mode after this gate?
>
> **NEVER offer Implementation until Gate 5c (TaskPlanning) is complete.**

Proceed with the task using loaded context and mode-specific rules.

---

## Gates & Approvals

> [!CAUTION]
> **⛔ CRITICAL: You MUST load `${AGENT_ROOT}/modes/_gates.md` at the START of every planning session.**
>
> Failure to load and follow gates = **WORKFLOW VIOLATION**.

### Planning Chain (MEMORIZE THIS)

```
ProductSpecReview → [DesignAnalysis] → FeaturePlanning → [ProductRoadmap] → [TechStackDecision] → TechSpec → TaskPlanning → Implementation → Verification → PR
     GATE 1          GATE 2 (opt)        GATE 3          GATE 3.25          GATE 3.5           GATE 4    GATES 5a-5c      EXECUTION      GATE 5.9    GATE 6
```

**Gate Checkpoints**:

| Gate | Mode | Artifact | Next Action |
|------|------|----------|-------------|
| 1 | ProductSpecReview | Gap Analysis | → FeaturePlanning (or DesignAnalysis if Figma) |
| 2 | DesignAnalysis | Design Review | → FeaturePlanning |
| 3 | FeaturePlanning | Epic | → ProductRoadmap (new project) OR TechSpec (existing) |
| 3.25 | ProductRoadmap | Roadmap Summary | → TechStackDecision |
| 3.5 | TechStackDecision | Stack Selection | → TechSpec |
| 4 | TechSpec | Tech Spec | → TaskPlanning |
| 5a-5c | TaskPlanning | Tasks + Readiness | → User selects first task |
| 5.9 | Verification | Evidence in `.verification/` | → Pull Request |
| 6 | Implementation | PR Description | → Code Review |

**Standard Approval Format**:
```
✅ **[Mode Name] Complete**
- **Artifact**: [ID] (path or URL)
- **Summary**: [1-line description]
- **Next Step**: [MANDATORY next mode from chain above]

> **⏸️ APPROVAL REQUIRED**: Reply with `Approve` to proceed.
```

> [!WARNING]
> **After approval, you MUST immediately proceed to the NEXT mode in the chain.**
> Do NOT ask "should I proceed to implementation" until Gate 5c is complete.

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
