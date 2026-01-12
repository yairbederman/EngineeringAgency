# Workflow Gate Checkpoints

> **Core Principle**: Each planning step requires human approval before proceeding.
> Entry conditions for **Implementation** and **BugFix** are defined in `core-rules.md`.

> [!CAUTION]
> **⛔ GATE ENFORCEMENT IS NON-NEGOTIABLE**
> 
> Every gate marked with "STOP" is a **HARD STOP**. The agent:
> - **MUST NOT** proceed to the next mode without explicit user approval
> - **MUST NOT** create artifacts for the next phase until approval is received
> - **MUST NOT** generate mockups, designs, or code beyond the current gate
> - **MUST** display the Standard Approval Format and wait for user response
> - **MUST** treat any attempt to bypass gates as a **workflow violation**
> 
> **Valid approval responses**: `Approve`, `Yes`, `Proceed`, or mode-specific options listed in each gate.
> **Any other response**: Treat as feedback requiring revision, NOT as approval.

---

## ⚡ CRITICAL: Planning Phase Chaining Rule

> [!IMPORTANT]
> **The Planning Phase is a SINGLE CHAIN that MUST complete before Implementation.**
>
> ```
> ProductSpecReview → [DesignAnalysis] → FeaturePlanning → [ProductRoadmap] → [TechStackDecision] → TechSpec → TaskPlanning → Implementation → Verification → PR
>       GATE 1          GATE 2 (opt)        GATE 3          GATE 3.25          GATE 3.5           GATE 4    GATES 5a-5c      EXECUTION      GATE 5.9    GATE 6
> ```
>
> **Gate 2 is OPTIONAL**: Only triggers if Figma links are present in the Product Spec.
> **Gate 3.25 is CONDITIONAL**: Only triggers if Epic creates a new project/codebase.
> **Gate 3.5 is CONDITIONAL**: Only triggers if Epic creates a new project/codebase.
>
> **After each approval, you MUST immediately proceed to the next planning phase.**
> Do NOT ask "should I proceed to implementation" until ALL planning gates (through Gate 5c) are complete.
>
> | Gate Approved | Next Action (MANDATORY) |
> |---------------|-------------------------|
> | Gate 1 (ProductSpecReview) | → Proceed to **DesignAnalysis** (if Figma) OR **FeaturePlanning** (if no Figma) |
> | Gate 2 (DesignAnalysis) | → Proceed to **FeaturePlanning** |
> | Gate 3 (FeaturePlanning) | → Proceed to **ProductRoadmap** (if new project) OR **TechSpec** (if existing project) |
> | Gate 3.25 (ProductRoadmap) | → Proceed to **TechStackDecision** with roadmap context |
> | Gate 3.5 (TechStackDecision) | → Proceed to **TechSpec** with chosen technology |
> | Gate 4 (TechSpec) | → Proceed to **TaskPlanning** |
> | Gate 5c (TaskPlanning) | → **STOP** - User selects first task for Implementation |

---

## ⚡ Change Request Handling at Gates

> [!IMPORTANT]
> **Mid-Flow Change Protocol**
>
> If user introduces a change request at ANY gate:
> 1. **STOP** current gate processing
> 2. **Load** `${AGENT_ROOT}/modes/planning/change-request.md`
> 3. **Execute** change request protocol
> 4. **Resume** at current gate (not restart from beginning)

### Phase-Specific Behavior

| Current Phase | Change Detected | Action |
|---------------|-----------------|--------|
| PLANNING_SPEC | During ProductSpecReview | Update spec inline, no mode switch |
| PLANNING_EPIC | After Epic approved | Switch to ChangeRequest, update Epic |
| PLANNING_TECH | After TechSpec approved | Switch to ChangeRequest, flag architecture impact |
| PLANNING_TASKS | After Tasks created | Switch to ChangeRequest, mark affected tasks |
| EXECUTION | Task in progress | **⚠️ CRITICAL** - Flag code impact, estimate rework |
| COMPLETION | PR submitted | **🔴 SCOPE CREEP** - May need new Epic |

### Gate Re-Approval Rules

| Change Type | Re-Approval Required |
|-------------|---------------------|
| Minor | No - Continue at current gate |
| Scope | Yes - Re-approve current + affected downstream |
| Major | Yes - Re-approve ALL from ProductSpec forward |

### Rollback Trigger

If user says "go back", "undo", or "revert":
1. Check `.versions/` for previous artifact versions
2. Present version list with dates
3. On selection:
   - Call `storage.rollbackArtifact()`
   - Cascade rollback to downstream if needed
   - Resume at appropriate gate

### Change Handling at Each Gate

| Gate | On Change Request |
|------|-------------------|
| Gate 1 | Update ProductSpec, re-present Gap Analysis |
| Gate 2 | Update Design Review if design affected |
| Gate 3 | Update Epic, regenerate acceptance criteria |
| Gate 3.5 | Re-evaluate tech stack decision if architectural |
| Gate 4 | Update TechSpec, flag implementation changes |
| Gate 5a-5c | Update tasks, mark affected as needs-review |
| Gate 5.9 | Re-run verification after code changes |

## Gate 1: After ProductSpecReview

| Attribute | Value |
|-----------|-------|
| **Artifact** | Gap Analysis Report |
| **Trigger** | ProductSpecReview mode completes |

**Action**: If Critical Gaps exist, present severity-segmented questions:

**Severity Segmentation (MANDATORY)**:
- 🔴 **BLOCKER**: Prevents implementation (missing logic, contradictory requirements)
- 🟠 **HIGH RISK**: Likely to cause rework (ambiguous edge cases)
- 🟢 **LOW RISK**: Minor clarifications

**User Options**:
- **Option A**: Post questions to Confluence and wait for PM response
- **Option B**: User provides answers directly in chat
- **Option C**: Proceed with provisional assumptions (log in Epic's "Assumptions Log")

**Gate**: STOP until user selects an option.

**On Approval**: → Immediately proceed to **FeaturePlanning** (or **DesignAnalysis** if Figma links exist)

---

## Gate 2: After DesignAnalysis

| Attribute | Value |
|-----------|-------|
| **Artifact** | Design Review Report |
| **Trigger** | Product Spec contains Figma links |
| **Skip Condition** | No Figma links exist (user may approve skip) |

**Action**:
- Extract component tree and layout properties from Figma
- Map Figma tokens to project design system
- Identify responsive variants and breakpoint differences
- List required assets (images, icons needing export)
- Match Figma component instances to project components
- Flag design gaps (missing states, incomplete structures)

**Output**: Design Review Report with Token Mapping, Component Reuse Checklist, Responsive Summary, Asset Manifest.

**Gate**: STOP until user approves Design Review.

**On Approval**: → Immediately proceed to **FeaturePlanning**

---

## Gate 3: After FeaturePlanning

| Attribute | Value |
|-----------|-------|
| **Artifact** | Jira Epic (`atlassian`) OR `epic.md` (`local`) |
| **Trigger** | FeaturePlanning mode completes |

**Action**:
- Create Epic via `storage.createEpic()`
- Update Product Spec's Links section (Atlassian mode only)
- If assumptions logged, Epic must include "Assumptions Log" section and `needs-validation` label

**Gate**: STOP until user approves Epic.

**On Approval** (CONDITIONAL BRANCHING):
- **If Epic creates a NEW project/codebase**: → Proceed to **ProductRoadmap** (Gate 3.25)
- **If Epic modifies EXISTING project**: → Skip to **TechSpec** (Gate 4)

> [!IMPORTANT]
> For new projects, ProductRoadmap is **MANDATORY** to ensure tech stack decisions consider future needs.

---

## Gate 3.25: Product Roadmap Analysis (Conditional)

| Attribute | Value |
|-----------|-------|
| **Artifact** | Product Roadmap Summary |
| **Trigger** | Epic involves **new project/codebase** creation |
| **Skip Condition** | Epic only modifies existing project(s) OR user explicitly says "no future plans" |

**Action**:
- Present Future Feature Discovery questions (CMS, user accounts, e-commerce, integrations, scale)
- Document Product Roadmap Summary with phases and technical implications
- Determine tech stack direction based on future needs

**Questions to Ask**:
1. CMS/blog for content updates?
2. User accounts and login?
3. E-commerce or payments?
4. Third-party integrations?
5. Expected traffic scale?
6. Who maintains after launch?

**Gate**: STOP until user answers roadmap questions OR replies `Skip`.

**On Completion**: 
- Document roadmap in Epic's Decisions Log
- → Proceed to **TechStackDecision** (Gate 3.5) with roadmap context

---

## Gate 3.5: Tech Stack Decision (Conditional)

| Attribute | Value |
|-----------|-------|
| **Artifact** | Tech Stack Options Table |
| **Trigger** | Epic involves **new project/codebase** creation |
| **Skip Condition** | Epic only modifies existing project(s) with established tech stack |

**Action**:
- Present 2-4 viable tech stack options with pros/cons
- Include recommendation with justification
- Get explicit user approval before writing detailed Tech Spec

**User Options**:
- Select Option A, B, C, etc.
- Request more options or alternatives

**Gate**: STOP until user selects a tech stack option.

**On Approval**: 
- Document decision in Epic's Decisions Log
- → Proceed to generate detailed **TechSpec** with chosen technology

---



## Gate 4: After TechSpec

| Attribute | Value |
|-----------|-------|
| **Artifact** | Jira Task (`atlassian`) OR `tech-spec.md` (`local`) |
| **Trigger** | TechSpec mode completes |

**Action**:
- Present generated Tech Spec to user
- **MANDATORY**: Ask user to choose:
  - `Yes, Inject & Proceed`: Create via `storage.createSpec()`
  - `No, Local Only`: Keep local artifact (no storage write)
- **Start Condition**: Create artifact ONLY if "Yes" selected

**Gate**: STOP until user makes a decision.

**On Approval**: → Immediately proceed to **TaskPlanning** (do NOT offer implementation yet)

---

## Gate 5: After TaskPlanning (3 Sequential Gates)

### Gate 5a: Presentation Gate (Step 7)

| Attribute | Value |
|-----------|-------|
| **Artifact** | Proposed task list with dependency order |
| **Trigger** | Task list generated |

**Validation**: Apply Pre-Creation Validation from `planning/_validation-checklist.md`

**Gate**: STOP → User approves task list before Jira creation.

### Gate 5b: Pre-Implementation Gate (Step 10)

| Attribute | Value |
|-----------|-------|
| **Artifact** | Traceability Matrix + Technical Readiness Checklist |
| **Trigger** | Jira tasks created and links verified |

**Action**:
- Generate Epic → Tech Spec → Task traceability matrix
- Verify scope boundaries (In Scope vs Out of Scope)
- Complete technical readiness checklist

**Gate**: STOP → User confirms scope before implementation.

### Gate 5c: Final Readiness (Step 11)

| Attribute | Value |
|-----------|-------|
| **Artifact** | Summary with task keys, layer breakdown, dependency order |
| **Trigger** | All tasks created and linked |

**Gate**: STOP → User selects first task to implement (do NOT auto-proceed).

---

## Gate 5.9: Live Verification Gate (MANDATORY)

> [!CAUTION]
> **⛔ NO TASK IS COMPLETE WITHOUT VERIFICATION**
>
> This gate blocks task completion until implementation is verified via browser/MCP tools.
> **Position**: This gate MUST complete BEFORE Gate 6 (Pull Request).

| Attribute | Value |
|-----------|-------|
| **Artifact** | Verification evidence in `.verification/[TaskKey]/` |
| **Trigger** | Code committed, before status update to "In Review" |

**Load**: `${AGENT_ROOT}/modes/execution/verification-gate.md`

### Scope Detection

> **Single Source of Truth**: See `verification-gate.md` § "Scope Detection" for authoritative table.
> The scope detection logic and file patterns are defined there to prevent drift.

**Summary**:
| Scope | Verification | User Approval |
|-------|--------------|---------------|
| DOC_ONLY | Auto-approve | ✅ |
| BUILD_CHECK | Build pass | ⚠️ |
| API_TEST / BROWSER_VISUAL / FULL_STACK | Full verification | ❌ Required |

### Action

1. Detect verification scope from `git diff --name-only`
2. Execute appropriate verification protocol (per `verification-gate.md`)
3. Capture evidence (screenshots, recordings, logs)
4. Present results with checklist completion status

### Gate Behavior

- **DOC_ONLY**: Auto-approve, proceed to Step 3.3
- **BUILD_CHECK**: Require build pass, then proceed
- **API_TEST / BROWSER_VISUAL / FULL_STACK**: STOP → User approval required

### User Options

- `Approve` → Mark task complete, proceed to PR
- `Retry` → Re-run verification
- `Skip [reason]` → Bypass with justification (adds `unverified` label)

### On Skip

1. Log justification in task comment
2. Add `unverified` label to task
3. Include warning in PR description

---

## Gate 6: After Implementation/BugFix

| Attribute | Value |
|-----------|-------|
| **Artifact** | PR Description with self-review checklist |
| **Trigger** | Gate 5.9 (Verification) passed, Jira status updated to "In Review" |

**Action**:
- Complete self-review checklist
- Generate PR description with context
- Detect breaking changes
- Provide reviewer guidance

**Output**: Ready-to-submit PR with Summary, Linked Jira, Screenshots/recordings (Frontend), Test evidence, Breaking change warnings, Rollback plan.

**Gate**: STOP → User creates PR or requests revisions.

---

## Standard Approval Format

After each mode completes, use this format:

```
✅ **[Mode Name] Complete**
- **Artifact**: [ID] (URL or path based on storage backend)
- **Summary**: [1-line description]
- **Next Step**: [Next mode name]

> **⏸️ APPROVAL REQUIRED**: Reply with `Approve` to proceed.
```

---

## Artifact Display Protocol

> **Purpose**: Consistent artifact display regardless of storage backend.

| Backend | Epic Display | Tech Spec Display | Task Display |
|---------|--------------|-------------------|--------------|
| `atlassian` | `[PROJ-123](jira-url)` | `[PROJ-124](jira-url)` | `[PROJ-125](jira-url)` |
| `local` | `[EPIC-001](file:///.specs/.../epic.md)` | `[tech-spec](file:///.../tech-spec.md)` | `[TASK-001](file:///.../TASK-001.md)` |
