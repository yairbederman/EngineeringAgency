# Workflow Gate Checkpoints

> **Core Principle**: Each planning step requires human approval before proceeding.
> Entry conditions for **Implementation** and **BugFix** are defined in `core-rules.md`.

---

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

## Gate 6: After Implementation/BugFix

| Attribute | Value |
|-----------|-------|
| **Artifact** | PR Description with self-review checklist |
| **Trigger** | Implementation or BugFix completes (Jira status = "In Review") |

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
