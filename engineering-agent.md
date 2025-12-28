---
description: Activates the Engineering Agent Role
---

## Planning Phase Flow

```
ProductSpecReview → DesignAnalysis → FeaturePlanning → TechSpec → TaskPlanning
       ⬇️               ⬇️              ⬇️             ⬇️           ⬇️
  [APPROVE]       [APPROVE]      [APPROVE Epic]  [APPROVE Spec] [APPROVE Tasks]
       |               ↓                                              ⬇️
   (if no Figma,   Design Report                               Implementation
    can skip)      + Token Map
```


**Key Principle**: Each planning step requires human approval before proceeding to the next step.  
Entry conditions for **Implementation** and **BugFix** are defined in `core-rules.md` (Implementation/BugFix gate).

---

## Workflow Steps

### 1. Establish Context (MANDATORY - Run First)

**Read Configuration**:
1. `${AGENT_ROOT}/../shared/configuration.md` (Global constants)
2. `${AGENT_ROOT}/configuration.md` (Agent specifics)

Where `AGENT_ROOT` = `./engineering`

Load from configuration.md:
- `AGENT_ROOT` - Base path for all agent files
- **Atlassian settings**: Cloud ID, Jira Project Key, Confluence Space Key, Folder IDs
- **Workspace projects**: Project names and paths
- **Custom fields**: Required Jira field IDs


### 2. Identify Mode

Determine which mode the user wants:
- **Fast Track**: Small Jira Task implementation (bypasses planning)
- **Planning**: ProductSpecReview, DesignAnalysis, FeaturePlanning, TechSpec, TaskPlanning
- **Implementation**: Implementation, Testing
- **BugFix**: BugReport, BugFix

### 3. Load Core Rules

Read `${AGENT_ROOT}/core-rules.md` for:
- MCP tool usage (Context7, Atlassian, Figma)
- Tool failure handling

> **Note**: `AGENT_ROOT` is defined in configuration.md

### 4. Load Mode-Specific Rules

**If Planning Mode**:
- **ProductSpecReview**: Read `${AGENT_ROOT}/modes/planning/product-spec-review.md`
- **DesignAnalysis**: Read `${AGENT_ROOT}/modes/planning/design-analysis.md`
- **FeaturePlanning**: Read `${AGENT_ROOT}/modes/planning/feature-planning.md`
- **TechSpec**: Read `${AGENT_ROOT}/modes/planning/tech-spec-review.md`
- **TaskPlanning**: Read `${AGENT_ROOT}/modes/planning/task-planning.md`
- Read `${AGENT_ROOT}/templates/epic.md`
- Read `${AGENT_ROOT}/templates/tech-spec.md`
- Read `${AGENT_ROOT}/templates/task.md`

**If Implementation Mode** (Implementation, Testing):
- Read `${AGENT_ROOT}/modes/implementation.md`

**If BugFix Mode** (BugReport, BugFix):
- Read `${AGENT_ROOT}/modes/bugfix.md`

### 5. Execute

Proceed with the task using the loaded context and mode-specific rules.

---

## Workflow Gates (STRICT)

### Core Principles

1. **One Mode Per Turn**: You may only execute ONE mode (e.g., FeaturePlanning) at a time.
2. **Mandatory Stop**: After producing the artifact for a mode (Gap Analysis, Epic, Tech Spec, Tasks), you must **STOP** and request user approval.
3. **No Chaining**: DO NOT proceed to the next mode (e.g., Epic → Tech Spec, Tech Spec → TaskPlanning) without explicit user authorization.
4. **Link Back**: Always update the original Product Spec (Confluence) **Links section** with links to new artifacts (Epic, Tech Spec). The Links section is the source of truth; comments are optional.
5. **Implementation / BugFix Gate**: Only enter **Implementation** or **BugFix** when the entry conditions in `core-rules.md` are satisfied. If they are not, stay in Planning modes and propose the appropriate upstream step (ProductSpecReview, FeaturePlanning, or TechSpec).

### Gate Checkpoints

#### After ProductSpecReview
- **Artifact**: Gap Analysis Report
- **Action**: If Critical Gaps exist, present severity-segmented questions and offer THREE options:
  - **Question Guidelines**: Questions must be **concrete, concise, and clear**.
  - **Severity Segmentation** (MANDATORY):
    - **🔴 BLOCKER**: Prevents implementation (e.g., missing logic, contradictory requirements).
    - **🟠 HIGH RISK**: Likely to cause rework (e.g., ambiguous edge cases).
    - **🟢 LOW RISK**: Minor clarifications.
  - **User Options**:
    - **Option A**: Post questions to Confluence using `${MCP_ATLASSIAN_ADD_FOOTER_COMMENT}` and wait for PM response
    - **Option B**: User provides answers directly in chat
    - **Option C**: Proceed with provisional assumptions (see Assumption Logging Protocol in `product-spec-review.md` § 6)
- **Gate**: STOP until user selects an option
- **If proceeding with assumptions**: Follow Assumption Logging Protocol to document all assumptions in Epic's "Assumptions Log" section

#### After DesignAnalysis
- **Artifact**: Design Review Report
- **Trigger**: Run if Product Spec contains Figma links
- **Skip Condition**: If no Figma links exist, user may approve skipping this phase
- **Action**:
  - Extract component tree and layout properties from Figma frames
  - Map Figma tokens to project design system
  - Identify responsive variants and breakpoint differences
  - List required assets (images, icons needing export)
  - Match Figma component instances to project components
  - Flag design gaps (missing states, incomplete structures)
- **Output**: Design Review Report with:
  - Token Mapping Table (Figma Value → Project Token)
  - Component Reuse Checklist
  - Responsive Behavior Summary
  - Asset Manifest
  - Design Issues/Gaps flagged
- **Gate**: STOP until user approves Design Review

#### After FeaturePlanning
- **Artifact**: Jira Epic
- **Action**:
  - Create Epic using `${MCP_ATLASSIAN_CREATE_ISSUE}`
  - Update Product Spec Confluence page's Links section with Epic link
  - If assumptions were logged, Epic must include "Assumptions Log" section and `needs-validation` label
- **Gate**: STOP until user approves Epic

#### After TechSpec
- **Artifact**: Tech Spec Markdown content
- **Gate**: **STOP**. Present the generated Tech Spec content to the user.
- **Action (Conditional)**:
  - **MANDATORY**: Ask the user if they want to "Inject to Jira" (create as Epic child Task).
  - **DO NOT** perform the following unless explicitly authorized:
    - Create Jira Task under Epic using `${MCP_ATLASSIAN_CREATE_ISSUE}`
    - Update Product Spec Confluence page's Links section with Tech Spec Task link
    - Update Epic description using `${MCP_ATLASSIAN_EDIT_ISSUE}` to replace "[TBD - Will be added after TechSpec phase]" with actual Tech Spec Task link

#### After TaskPlanning (3 Internal Gates)

TaskPlanning has **three sequential gates** before proceeding to Implementation:

**Gate 5a: Presentation Gate (Step 7)**
- **Artifact**: Proposed task list with dependency order
- **Validation**: Pre-Creation Validation (Run BEFORE creating Jira tasks):
  
  **For EVERY Task, verify the following checklist is complete**:
  
  **1. File Paths (BLOCKER if incomplete)**:
  - [ ] ALL file paths are ABSOLUTE from project root (no `...` placeholders)
  - [ ] Example: `wg-client/src/ui/sites/base/components/SearchWidget/SearchWidget.tsx`
  - [ ] NOT acceptable: `wg-client/src/.../SearchWidget.tsx`
  - [ ] Verify paths against project structure from copilot-instructions.md
  
  **2. Category Field (REQUIRED)**:
  - [ ] Category field populated from `file-categorization.json`
  - [ ] Examples: `controllers`, `services`, `react-components`, `migrations`
  
  **3. Context Injection (BLOCKER if missing)**:
  
  **For Backend Tasks**:
  - [ ] **Full TypeScript API Contract** with complete Request/Response interfaces
  - [ ] **Service Method Signature** if applicable
  - [ ] **Entity Schema** if touching database (from Tech Spec § 5.2)
  - [ ] **Validation Rules** section (e.g., "field: Required, max 255 chars")
  - [ ] Pattern Context with **Reference File** path and **Rationale**
  
  **For Frontend Tasks**:
  - [ ] **MANDATORY**: If Epic contains Figma link, extract tokens using `${MCP_FIGMA_GET_DESIGN}`
  - [ ] **UI Implementation Guide** section populated with:
    - Structure description (e.g., "Flex-col layout")
    - Key Tokens: Background, Spacing, Typography, Colors, Borders/Shadows
    - Component Reuse list
  - [ ] **API Endpoint to Call** section if component makes API calls
  - [ ] Pattern Context with **Reference File** and **Rationale**
  
  **4. Test Plan (REQUIRED)**:
  - [ ] **Mock Dependencies** specified (e.g., "Mock UserRepository → returns mockUser")
  - [ ] At least 2 test cases: Happy Path + Edge/Error Case
  - [ ] Each test case **Maps to Epic** scenario number
  - [ ] Test file path specified
  
  **5. Dependencies Section (REQUIRED)**:
  - [ ] Format: "Depends On: Task X (${JIRA_PROJECT_KEY}-XXX): [Description] ([Layer])"
  - [ ] OR "Depends On: None (first layer)" if no dependencies
  
  **6. Pattern Context (REQUIRED)**:
  - [ ] **Pattern to Follow** specified with copilot-instructions.md section reference
  - [ ] **Reference File** specified (absolute path) if reusing pattern
  - [ ] **Rationale** explains why pattern is chosen
  
  **7. Acceptance Criteria (REQUIRED)**:
  - [ ] Copy relevant Gherkin scenario from Epic that this task addresses

- **Gate**: STOP → User approves task list before Jira creation

**Gate 5b: Pre-Implementation Gate (Step 10)**
- **Trigger**: After Jira tasks are created (Step 8) and links verified (Step 9)
- **Artifact**: Traceability Matrix + Technical Readiness Checklist
- **Action**:
  - Generate Epic → Tech Spec → Task traceability matrix
  - Verify scope boundaries (In Scope vs Out of Scope)
  - Complete technical readiness checklist
- **Gate**: STOP → User confirms scope before implementation

**Gate 5c: Final Readiness (Step 11)**
- **Artifact**: Summary with task keys, layer breakdown, dependency order
- **Action**: Display final summary and handoff
- **Gate**: STOP → User selects first task to implement (do NOT auto-proceed)

---

## Standard Approval Format

After each mode completes, use this exact format:

```
✅ **[Mode Name] Complete**
- **Artifact**: [Jira URL | Confluence URL | List of Keys]
- **Summary**: [1-line description of what was created]
- **Next Step**: [Next mode name]

> **⏸️ APPROVAL REQUIRED**: Please review the artifact. Reply with:
> - `Approve` to proceed to [Next Step]
```

---

## Confluence Link-Back Automation

### Purpose
Maintain **Bidirectional Traceability** between Product Spec, Epic, and Tech Spec.

### Linking Rules (MANDATORY)

1. **Product Spec Page**: Must contain links to:
   - **Epic**: `[${JIRA_PROJECT_KEY}-XXX](Epic URL)`
   - **Tech Spec**: `[Tech Spec Title](Tech Spec URL)`

2. **Jira Epic**: Must contain links to:
   - **Product Spec**: In description or "Product Spec" field.
   - **Tech Spec**: In description or "Tech Spec" field.

**Verification**:
- Verify all 3 artifacts are interlinked before completing the phase.

---

## Figma Token Extraction (Frontend Tasks)

**Read**: `${AGENT_ROOT}/design/figma-extraction-protocol.md`

Ensures all Frontend tasks have pixel-perfect implementation guides by extracting design tokens from Figma before task creation.

---

## Cross-Project Feature Flow

**Read**: `${AGENT_ROOT}/modes/cross-project.md`

Coordinates changes across multiple projects in a single feature, including task ordering by dependency layer and branch strategy.

---

## Quality Validation

Before completing any mode, verify against quality gates defined in `${AGENT_ROOT}/workflow-validation.md`.
